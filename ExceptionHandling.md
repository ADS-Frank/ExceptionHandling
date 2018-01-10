# Exception Handling Do's Vs. Don'ts


## Introduction

This file contains Do's and Don'ts of exception handling. Each rule contains
an example for a proper use, and an example for improper use.


## Table of Contents
1. [Use Specific Exceptions](#e1)
2. [Wrap Underlying Exceptions](#e2)
3. [Avoid Runtime Exceptions](#e3)
4. [Document Exceptions](#e4)


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
        throw new NullStringException("method expects a string who's value is not null.");
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
        if (ssn == null) throw SSNNullException("The ssn passed in was null.");

        if (ssn.length() != 9) throw SSNLengthException("The length of the SSN was not 9.");

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
class InvalidSSNException extends RuntimeException { // <- unchecked by the compiler
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
public Account retrieveAccount(int accountNumber) throws InvalidInputException, SystemUnavailableException {
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

DONT - Undocument exceptions in methods.  This will force the programmer to understand the code that they are calling, break enapsulation, add cognitive overhead, and slow down the programmer.

```java
// Missing a docstring!
public Account retrieveAccount(int accountNumber) throws InvalidInputException, SystemUnavailableException {
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