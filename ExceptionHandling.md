# Exception Handling Do's Vs. Don'ts


## Introduction

This file contains Do's and Don'ts of exception handling. Each rule contains
an example for a proper use, and an example for improper use.


## Table of Contents
1. [Use Specific Exceptions](#e1)
2. [Wrap Underlying Exceptions](#e2)
3. [Avoid Runtime Exceptions](#e3)


### Use Specific Exceptions <a name="e1"></a>

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
        throw new Exception("method expects a string who's value is not null.");
    }

    return str.toUpperCase();
}
```


### Wrap Underlying Exceptions <a name="e2"></a>

DO - Wrap underlying exceptions which may be thrown by the code.  This will attach a business meaning to them and make the code more maintainable.

```java
/**
 * Retrieves the last four digits of the SSN as an integer.
 *
 * @param ssn - The SSN to get the last four digits of.
 *
 * @return - The last four digits.
 *
 * @throws InvalidSSNFormatException - If the SSN is not a valid SSN.
 */
public int getLastFourDigitsOfSSN(String ssn) throws InvalidSSNException {
    try {
        return Integer.valueOf(ssn) % 1000;

    } catch (NumberFormatException nfe) {
        throw new InvalidSSNException("The ssn format was not valid.", nfe);
    }
}
```

DONT - Throw base exceptions.  These provide not extra meaning as to why the exception was thrown.

```java
/**
 * Retrieves the last four digits of the SSN as an integer.
 *
 * @param ssn - The SSN to get the last four digits of.
 *
 * @return - The last four digits.
 *
 * @throws InvalidSSNFormatException - If the SSN is not a valid SSN.
 */
public int getLastFourDigitsOfSSN(String ssn) throws NumberFormatException {
    try {
        return Integer.valueOf(ssn) % 1000;

    } catch (NumberFormatException nfe) {
        throw nfe;
    }
}
```

### Avoid Runtime Exceptions

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
 * @throws InvalidSSNFormatException - If the SSN is not a valid SSN.
 */
public int getLastFourDigitsOfSSN(String ssn) throws InvalidSSNException {
    try {
        return Integer.valueOf(ssn) % 1000;

    } catch (NumberFormatException nfe) {
        throw new InvalidSSNException("The ssn format was not valid.", nfe);
    }
}
```

DONT - Never throw unchecked exceptions.

```java
/**
 * ... Class Description Here ...
 */
class InvalidSSNException extends RuntimeException {
    // ...
}

/**
 * Retrieves the last four digits of the SSN as an integer.
 *
 * @param ssn - The SSN to get the last four digits of.
 *
 * @return - The last four digits.
 *
 * @throws InvalidSSNFormatException - If the SSN is not a valid SSN.
 */
public int getLastFourDigitsOfSSN(String ssn) {
    try {
        return Integer.valueOf(ssn) % 1000;

    } catch (NumberFormatException nfe) {
        throw new InvalidSSNException("The ssn format was not valid.", nfe);
    }
}
```