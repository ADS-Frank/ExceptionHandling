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
9. [Prevent Normal Execution on Error](#e9)
10. [Push Error Handling Responsibility Upwards](#e10)
11. [Logging and Throwing](#e11)
12. [Always include Message](#e12)
13. [Treating Runtime Exceptions](#e13)
14. [Exception Logging](#e14)
15. [Finally Blocks](#e15)
16. [RuntimeExceptions in the JDK](#e16)


### <a name="e1"></a> Use Specific Exceptions

DO - Throw exception instances that are specific and descriptive of the underlying reason for why an exception was thrown.

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

DONT - Throw exception instances that are general such as ```Exception``` or ```NullPointerException```.  Without inspecting the code, they provide the user of the code with no understanding of why an exception was thrown.

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

DO - Wrap underlying exceptions which may be thrown by the code.  This will attach a business meaning to them and make the code more maintainable.

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

DONT - Throw base exceptions caught in the program.  These provide not extra meaning as to why the exception was thrown.

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

DO - Always throw checked exceptions.  The compiler will prevent the code from compiling if an exception case is not properly handled.

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

DONT - Throw unchecked exceptions.  These are unchecked by the compiler
and thus without the programmers knowledge, could change the execution of the program.

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

// Below is in some other class

/**
 * Gets the user from the database.
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
 */
public void displayUser(byte[] accountNumber) {
    User user = otherClass.retrieveUser(accountNumber);
    if (user != null) {
        setFormValue(user.getName());
    }
}

// Below is in some other class.

/**
 * Gets the user from the database.
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
 * This method assumes that url is not null.
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

### <a name="e9"></a> Prevent Normal Execution on Error
### <a name="e10"></a> Push Error Handling Responsibility Upwards
### <a name="e11"></a> Logging and Throwing
### <a name="e12"></a> Always include Message
### <a name="e13"></a> Treating Runtime Exceptions
### <a name="e14"></a> Exception Logging
### <a name="e15"></a> Finally Blocks
### <a name="e16"></a> RuntimeExceptions in the JDK
