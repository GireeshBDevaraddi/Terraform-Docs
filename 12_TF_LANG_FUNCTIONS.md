# FUNCTIONS
**Reference - https://developer.hashicorp.com/terraform/language/functions**
- The Terraform language includes a number of built-in functions that you can call from within expressions to transform and combine values. The general syntax for function calls is a function name followed by comma-separated arguments in parentheses
`> max(5, 12, 9)  -> 12`
- The Terraform language does not support user-defined functions, and so only the functions built in to the language are available for use
## Numeric Functions
1. `abs` - abs returns the absolute value of the given number. In other words, if the number is zero or positive then it is returned as-is, but if it is negative then it is multiplied by -1 to make it positive before returning it.
2. `ceil` - ceil returns the closest whole number that is greater than or equal to the given value, which may be a fraction.
3. `floor` - returns the closest whole number that is less than or equal to the given value, which may be a fraction
4. `log`-  returns the logarithm of a given number in a given base
5. `max` - takes one or more numbers and returns the greatest number from the set.
6. `min` - takes one or more numbers and returns the smallest number from the set.
7. `parseint` - parses the given string as a representation of an integer in the specified base and returns the resulting number. The base must be between 2 and 62 inclusive. If the given string contains any non-digit characters or digit characters that are too large for the given base then parseint will produce an error
    ```
    > parseint("100", 10) -> 100
    > parseint("FF", 16)    ->255
    ```
8. `pow` - calculates an exponent, by raising its first argument to the power of the second argument
9. `signum` - determines the sign of a number, returning a number between -1 and 1 to represent the sign.
## String Functions
1. 
