# `short` and `java.lang.Short`

A primitive type:

- is not an object;
- has no methods;
- cannot be dereferenced with `.`.

## valueOf(short): Short

```java
    /**
     * Returns a {@code Short} instance representing the specified
     * {@code short} value.
     * If a new {@code Short} instance is not required, this method
     * should generally be used in preference to the constructor
     * {@link #Short(short)}, as this method is likely to yield
     * significantly better space and time performance by caching
     * frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  s a short value.
     * @return a {@code Short} instance representing {@code s}.
     * @since  1.5
     */
    public static Short valueOf(short s) {
        final int offset = 128;
        int sAsInt = s;
        if (sAsInt >= -128 && sAsInt <= 127) { // must cache
            return ShortCache.cache[sAsInt + offset];
        }
        return new Short(s);
    }
```



## parseShort(String): short

```java
/**
 * Parses the string argument as a signed decimal {@code
 * short}. The characters in the string must all be decimal
 * digits, except that the first character may be an ASCII minus
 * sign {@code '-'} ({@code '\u005Cu002D'}) to indicate a
 * negative value or an ASCII plus sign {@code '+'}
 * ({@code '\u005Cu002B'}) to indicate a positive value.  The
 * resulting {@code short} value is returned, exactly as if the
 * argument and the radix 10 were given as arguments to the {@link
 * #parseShort(java.lang.String, int)} method.
 *
 * @param s a {@code String} containing the {@code short}
 *          representation to be parsed
 * @return  the {@code short} value represented by the
 *          argument in decimal.
 * @throws  NumberFormatException If the string does not
 *          contain a parsable {@code short}.
 */
public static short parseShort(String s) throws NumberFormatException {
    return parseShort(s, 10);
}
```



The central rule is:

$$\boxed {\text{Java may box a method argument, but it does not box a primitive receiver before } .}$$
