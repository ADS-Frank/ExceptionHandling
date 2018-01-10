# Exception Handling Do's Vs. Don'ts

## Table of Contents
1. [Use Specific Exceptions](#e1)
2. [Wrap Underlying Exceptions](#e2)


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

DONT - Throw exception instances that are general such as ```Exception``` or ```NullPointerException```.

```java
/**
 * Returns the upper case version of the string.
 *
 * @param str - The string to upper case.
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