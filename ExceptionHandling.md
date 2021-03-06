# Exception Handling Do's Vs. Don'ts


## Introduction

This file contains Do's and Don'ts of exception handling. Each rule contains
an example for a proper use, and an example for improper use.


## Table of Contents
1. [Use Specific Exceptions](#e1)
2. [Wrap Underlying Exceptions](#e2)
3. [Avoid Runtime Exceptions](#e3)
4. [Document Exceptions](#e4)
5. [Catch Specific Exceptions](#e5)
6. [Caller Catches Exceptions](#e6)
7. [Assumptions of Input](#e7)
8. [Return Only Valid Things](#e8)
9. [Separate Exception Handling Code](#e9)
10. [Exception Logging and Alerting](#e11)
11. [Finally Blocks](#e15)


### <a name="e1"></a> Use Specific Exceptions

DO - Throw exception instances from public methods that are specific and descriptive of the underlying reason for why an exception was thrown.

DO - Define custom types to represent specific exceptions.

```java
/**
 * Returns the upper case version of the string.
 *
 * @param str - The string to upper case.
 * @return - The upper case version of the string.
 *
 * @throws NullStringException - if the string that was passed in was null.
 */
public String toUpperCase(String str) throws NullStringException {
    if (str == null) {
        throw new NullStringException(
            "method expects a string who's value is not null.");
    }

    return str.toUpperCase();
}
```

DONT - Throw exception instances that are general such as ```Exception``` or ```NullPointerException``` from public methods.  Without inspecting the code, they provide the user of the code with no understanding of why an exception was thrown.

```java
/**
 * Returns the upper case version of the string.
 *
 * @param str - The string to upper case.
 *
 * @return - The upper case version of the string.
 *
 * @throws Exception - if the string that was passed in was null.
 */
public String toUpperCase(String str) throws Exception {
    if (str == null) {
        // too general, could be any exception that can happen!
        throw new Exception("method expects a string who's value is not null.");
    }

    return str.toUpperCase();
}
```


### <a name="e2"></a> Wrap Underlying Exceptions

DO - Wrap underlying exceptions which may be thrown by public methods in the code if doing so will attach a business meaning to them and make the code more maintainable.

```java
/**
 * Retrieves the last four digits of the SSN as an integer.
 *
 * @param ssn - The SSN to get the last four digits of.
 *
 * @return - The last four digits.
 *
 * @throws InvalidSSNException - If the SSN is not a valid SSN.
 */
public int getLastFourDigitsOfSSN(String ssn) throws InvalidSSNException {
    try {
        return Integer.valueOf(ssn) % 1000;

    } catch (NumberFormatException nfe) {
        throw new SSNFormatException("The ssn format was not valid.", nfe);
    }
}
```

DONT - Throw base exceptions from public methods if they are ambiguous in meaning.

```java
/**
 * Retrieves the last four digits of the SSN as an integer.
 *
 * @param ssn - The SSN to get the last four digits of.
 *
 * @return - The last four digits.
 *
 * @throws Exception - If the SSN is not a valid SSN.
 */
public int getLastFourDigitsOfSSN(String ssn) throws NumberFormatException {
    try {
        return Integer.valueOf(ssn) % 1000;

    } catch (NumberFormatException nfe) {
        throw nfe; // <- no extra info known about why nfe was thrown
    }
}
```

### <a name="e3"></a> Avoid Runtime Exceptions

DO - Always throw checked exceptions from public methods.  The compiler will prevent the code from compiling if an exception case is not properly handled.

```java
/**
 * ... Class Description Here ...
 */
class InvalidSSNException extends Exception {
    // ...
}

/**
 * Retrieves the last four digits of the SSN as an integer.
 *
 * @param ssn - The SSN to get the last four digits of.
 *
 * @return - The last four digits.
 *
 * @throws InvalidSSNException - If the SSN is not a valid SSN.
 */
public int getLastFourDigitsOfSSN(String ssn) throws InvalidSSNException {
    try {
        return Integer.valueOf(ssn) % 1000;

    } catch (NumberFormatException nfe) {
        throw new InvalidSSNException("The ssn format was not valid.", nfe);
    }
}
```

DONT - Throw unchecked exceptions from public methods.  These are unchecked by the compiler and thus without the programmers knowledge, could change the execution of the program.

```java
/**
 * ... Class Description Here ...
 */
class InvalidSSNException extends RuntimeException { // <- unchecked by compiler
    // ...
}

/**
 * Retrieves the last four digits of the SSN as an integer.
 *
 * @param ssn - The SSN to get the last four digits of.
 *
 * @return - The last four digits.
 *
 * @throws InvalidSSNException - If the SSN is not a valid SSN.
 */
public int getLastFourDigitsOfSSN(String ssn) {
    try {
        return Integer.valueOf(ssn) % 1000;

    } catch (NumberFormatException nfe) {
        throw new InvalidSSNException("The ssn format was not valid.", nfe);
    }
}
```


### <a name="e4"></a> Document Exceptions

DO - Document all public methods with docstrings. In the docstrings add documentation with `@throws <exception-name> - <description>` to document all exceptions that can be thrown by a piece of code.

```java
/**
 * Retrieves an account from the database.
 *
 * @param accountNumber - the account number of the account to lookup.
 *
 * @return - The last four digits.
 *
 * @throws InvalidInputException - If the account number was not valid.
 *
 * @throws SystemUnavailableException - If the system was unavailable at the time of call.
 */
public Account retrieveAccount(int accountNumber)
        throws InvalidInputException, SystemUnavailableException {
    if (!valid(accountNumber)) {
        throw InvalidInputException(
            "The input account number was not a valid account number.");
    }

    try {
        return backend.retrieveFromDatabase(accountNumber);

    } catch (HttpResponseException hre) {
        throw new SystemUnavailableException(
            "The system was unavailable at this time.", hre);
    }
}
```

DONT - Undocument exceptions in methods.  This will force the programmer to understand the code that they are calling, break enapsulation, add cognitive overhead, and slow down the programmer.

```java
// Missing a docstring!
public Account retrieveAccount(int accountNumber)
        throws InvalidInputException, SystemUnavailableException {
    if (!valid(accountNumber)) {
        throw InvalidInputException("The input account number was not a valid account number.");
    }

    try {
        return backend.retrieveFromDatabase(accountNumber);

    } catch (HttpResponseException hre) {
        throw new SystemUnavailableException("The system was unavailable at this time.", hre);
    }
}
```

### <a name="e5"></a> Catch Specific Exceptions

DO - Use different exception handling blocks for different classes if they must be handled in different ways.

```java
/**
 * Ok button handler.
 */
public void onOkButtonClicked() {

    Input input = this.getInput();

    try {
        this.client.callServer(input);

    } catch (InputInvalidException iie) {
        messageUser("Input was invalid");

    } catch (ServerUnavailableException sue) {
        this.logger.logException(sue);
        messageUser("Server was unavailable at this time.");
    }
}
```

DONT - Handle exceptions in the same catch blocks that require different exception handling behavior.

```java
/**
 * Ok button handler.
 */
public void onOkButtonClicked() {

    Input input = this.getInput();

    try {
        this.client.callServer(input);

    } catch (InvalidInputException | ServerUnavailableException e) {
        // More error prone!  What if we accidentally performed actions for
        // one exception that we didnt actually want to?
        if (e instanceof InvalidInputException) {
            messageUser("Input was invalid");
        }
        if (e instanceof ServerUnavailableException) {
            this.logger.logException(e);
            messageUser("Server was unavailable at this time.");
        }
    }
}
```


### <a name="e6"></a> Caller Catches Exceptions

DO - The caller of a method should be responsible for any exception handling routines, no exception handling (logging/message displaying/etc.) should be done by callee code.

```java
/**
 * Displays the user's information on the screen.
 *
 * This method assumes that the accountNumber is valid.
 *
 * @param accountNumber - the account number of the user to display.
 */
public void displayUser(byte[] accountNumber) {
    try {
        User user = otherClass.retrieveUser(accountNumber);
        setFormValue(user.getName());
    } catch (ServerUnavailableException e) {
        logException(e);
        message("Server was unavailable ...");
    }
}

/**
 * Gets the user from the database.
 *
 * This method assumes that the accountNumber is valid.
 *
 * @param accountNumber - the account number of the user to display.
 *
 * @return - The user that was retrieved.
 *
 * @throws ServerUnavailableException - if the server was not reachable at the
 *     time of the method call.
 */
public User retrieveUser(byte[] accountNumber)
        throws ServerUnavailableException {
    try {
        Response resp = httpCall('/someUrl', toJson(accountNumber));
        return parseToUser(resp);
    } catch (HttpResponseException hre) {
        throw new ServerUnavailableException("could not reach /someUrl", hre);
    }
}
```

DONT - Attempt to handle an exception as a callee.  Predicting the future of how some code could be used is pretty hard.

```java
/**
 * Displays the user's information on the screen.
 *
 * This method assumes that the accountNumber is valid.
 *
 * @param accountNumber - the account number of the user to display.
 */
public void displayUser(byte[] accountNumber) {
    User user = otherClass.retrieveUser(accountNumber);
    if (user != null) {
        setFormValue(user.getName());
    }
}

/**
 * Gets the user from the database.
 *
 * This method assumes that the accountNumber is valid.
 *
 * @param accountNumber - the account number of the user to display.
 *
 * @return - The user that was retrieved.
 *
 * @throws ServerUnavailableException - if the server was not reachable at the
 *     time of the method call.
 */
public User retrieveUser(byte[] accountNumber)
        throws ServerUnavailableException {
    try {
        Response resp = httpCall('/someUrl', toJson(accountNumber));
        return parseToUser(resp);
    } catch (HttpResponseException hre) {
        log(hre);
        message("Server was unavailable...");
        return null;
    }
}
```

### <a name="e7"></a> Assumptions of Input

DO - Callee must document all assumptions made on the input that it recieves
from other methods.  If no documentation is specified, the caller should assume that any input is valid, and an exception will be thrown to handle bad input.

```java
/**
 * Displays the contents of a web page given a url.
 */
public void urlClicked(String url) {
    url = nullRemove(url);

    try {
        String response = httpCall(url);
        setBodyOfPage(response);

    } catch (PageNotFoundException e) {
        log(e);
        message("...");
    }
}

/**
 * HTTP Call method.
 *
 * This method assumes that url is not null.
 *
 * @param url - The url to retrieve.
 *
 * @return - the contents at that endpoint.
 *
 * @throws PageNotFoundException - the the page is not locatable.
 */
public String httpCall(String url) throws PageNotFoundException {
    String fullUrl = "http://" + url.toLowerCase();
    return doGET(fullUrl);
}
```

DON'T - Make undocumented assumptions of any data within a method.

```java
/**
 * Displays the contents of a web page given a url.
 */
public void urlClicked(String url) {
    try {
        // Notice! url could potentially be null.
        String response = httpCall(url);
        setBodyOfPage(response);

    } catch (PageNotFoundException e) {
        log(e);
        message("...");
    }
}

/**
 * HTTP Call method.
 *
 * @param url - The url to retrieve.
 *
 * @return - the contents at that endpoint.
 *
 * @throws PageNotFoundException - the the page is not locatable.
 */
public String httpCall(String url) throws PageNotFoundException {
    // Potential Runtime Exception Here!
    String fullUrl = "http://" + url.toLowerCase();
    return doGET(fullUrl);
}
```


### <a name="e8"></a> Return Only Valid Things

DO - Return only object that can be used by calling code.

```java
/**
 * Displays the contents of a web page given a url.
 */
public void urlClicked(String url) {
    url = nullRemove(url);

    try {
        String response = httpCall(url);
        setBodyOfPage(response);

    } catch (PageNotFoundException e) {
        log(e);
        message("...");
    }
}

/**
 * HTTP Call method.
 *
 * This method assumes that url is not null.
 *
 * @param url - The url to retrieve.
 *
 * @return - the contents at that endpoint.
 *
 * @throws PageNotFoundException - the the page is not locatable.
 */
public String httpCall(String url) throws PageNotFoundException {
    String fullUrl = "http://" + url.toLowerCase();
    return doGET(fullUrl);
}
```

DONT - Refrain from returning objects / data who's validity must be checked.

```java
/**
 * Displays the contents of a web page given a url.
 */
public void urlClicked(String url) {
    url = nullRemove(url);

    // No Documented Exceptions!  Without knowing caller will assume that
    // the return value was ok to use.
    String response = httpCall(url);

    // When knowing the caller has lost the ability to manipulate the exception
    // and react to their circumstance.
    if (response == null) {
        messageUser("...");
        return;
    }

    setBodyOfPage(response);
}

/**
 * HTTP Call method.
 *
 * @param url - The url to retrieve.
 *
 * @return - the contents at that endpoint.
 */
public String httpCall(String url) {
    String fullUrl = "http://" + url.toLowerCase();
    try {
        return doGET(fullUrl);
    } catch (PageNotFoundException e) {
        log(e);
        return null;
    }
}
```


### <a name="e9"></a> Separate Exception Handling Code

DO - Separate excetion handling code away from busines logic.

```java
/**
 * Routine for crediting an account.
 *
 * @param accountNumber - the account number to credit.
 *
 * @param amount - the amount to credit by.
 */
public void creditAccount(byte[] accountNumber, int amount) {
    try {
        beginTransaction();

        Account account = retrieveAccount(accountNumber);

        account.credit(amount);

        commitTransaction();

    } catch (AccountNumberNotFoundException annfe) {
        // ...
    } catch (AccountNumberNullException anne) {
        // ...
    } catch (TransactionRollbackException tre) {
        // ...
    } catch (InvalidBalanceException ibe) {
        // ...
    }
}
```

DONT - Mix exception handling code and busienss logic.

```java
/**
 * Routine for crediting an account.
 *
 * @param accountNumber - the account number to credit.
 *
 * @param amount - the amount to credit by.
 */
public void creditAccount(byte[] accountNumber, int amount) {
    beginTransaction();

    Account account = null;

    // Visually hard to follow. Execution could be trickey as well (Notice the
    // return statements within the catch blocks)
    try {
        account = retrieveAccount(accountNumber);
    } catch (AccountNumberNotFoundException annfe) {
        // ...
        return; // <- sneaky
    } catch (AccountNumberNullException anne) {
        // ...
        return; // <- sneaky
    }

    try {
        account.credit(amount);
    } catch (InvalidBalanceException ibe) {
        // ...
        return; // <- !
    }

    try {
        commitTransaction();
    } catch (TransactionRollbackException tre) {
        // ...
        return; // <- !
    }
}
```

### <a name="e11"></a> Logging and Alerting

DO - Only log exceptions once at the time the exception is finally handled.

DO - Add as much information regarding why the exception happened in order to make debugging easier.

DO - Alert the user with a message that contains no sensitive information about the program.

```java
/**
 * Displays the user's information on the screen.
 *
 * This method assumes that the accountNumber is valid.
 *
 * @param accountNumber - the account number of the user to display.
 */
public void displayUser(byte[] accountNumber) {
    try {
        User user = otherClass.retrieveUser(accountNumber);
        setFormValue(user.getName());
    } catch (ServerUnavailableException e) {
        logException(e);
        messageUser("Server was unavailable at this time. " +
            "Please try again at a later time.");
    }
}

/**
 * Gets the user from the database.
 *
 * This method assumes that the accountNumber is valid.
 *
 * @param accountNumber - the account number of the user to display.
 *
 * @return - The user that was retrieved.
 *
 * @throws ServerUnavailableException - if the server was not reachable at the
 *     time of the method call.
 */
public User retrieveUser(byte[] accountNumber)
        throws ServerUnavailableException {
    try {
        Response resp = httpCall('/someUrl', toJson(accountNumber));
        return parseToUser(resp);
    } catch (HttpResponseException hre) {
        throw new ServerUnavailableException("could not reach /someUrl", hre);
    }
}
```

DONT - Log an exception and then throw it to the next method.

DONT - Alert the user with the exception's message or any other sensitive information.

```java
/**
 * Displays the user's information on the screen.
 *
 * This method assumes that the accountNumber is valid.
 *
 * @param accountNumber - the account number of the user to display.
 */
public void displayUser(byte[] accountNumber) {
    try {
        User user = otherClass.retrieveUser(accountNumber);
        setFormValue(user.getName());
    } catch (ServerUnavailableException e) {
        logException(e);
        // What if this message has sensitive information?
        messageUser(e.getMessage());
    }
}

/**
 * Gets the user from the database.
 *
 * This method assumes that the accountNumber is valid.
 *
 * @param accountNumber - the account number of the user to display.
 *
 * @return - The user that was retrieved.
 *
 * @throws ServerUnavailableException - if the server was not reachable at the
 *     time of the method call.
 */
public User retrieveUser(byte[] accountNumber)
        throws ServerUnavailableException {
    try {
        Response resp = httpCall('/someUrl', toJson(accountNumber));
        return parseToUser(resp);
    } catch (HttpResponseException hre) {
        // How many exceptions does it look like were thrown?
        logException(hre);
        throw new ServerUnavailableException("could not reach /someUrl", hre);
    }
}
```


### <a name="e15"></a> Finally Blocks

DO - Use finally blocks to prevent resource leaks.

```java
/**
 * Gets the contents of a webPage.
 *
 * This method assumes that url is non null.
 *
 * @param url - the url to retireve the contents of.
 *
 * @return - the contents of the web page.
 *
 * @throws ServerAccessException - If the server is unavailable at the time
 *      the method call.
 *
 * @throws InvalidCharacterEncodingException - If the server returned content
 *      that we do not know how to decode.
 */
public String getContents(String url)
        throws ServerAccessException, InvalidCharacterEncodingException {

    HttpUrlConnection connection = null;

    try {
        connection = openConnection(url);

        return Utils.inputStreamToString(connection.getInputStream());

    } finally {
        if (connection != null) {
            connection.disconnect();
        }
    }
}
```

DONT - Try to account for resource closing in normal execution.

```java
/**
 * Gets the contents of a webPage.
 *
 * This method assumes that url is non null.
 *
 * @param url - the url to retireve the contents of.
 *
 * @return - the contents of the web page.
 *
 * @throws ServerAccessException - If the server is unavailable at the time
 *      the method call.
 *
 * @throws InvalidCharacterEncodingException - If the server returned content
 *      that we do not know how to decode.
 */
public String getContents(String url)
        throws ServerAccessException, InvalidCharacterEncodingException {

    HttpUrlConnection connection = openConnection(url);

    String response = Utils.inputStreamToString(connection.getInputStream());

    // if Utils.inputStreamToString throws an InvalidCharacterEncodingException
    // then we have a resource leak here
    connection.disconnect();

    return response;
}
```