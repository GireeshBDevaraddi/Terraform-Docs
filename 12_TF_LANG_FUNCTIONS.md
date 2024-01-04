# FUNCTIONS
** Reference - https://developer.hashicorp.com/terraform/language/functions **
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
1. `chomp` removes newline characters at the end of a string.
2. `endswith` takes two values: a string to check and a suffix string. The function returns true if the first string ends with that exact suffix - `endswith(string, suffix)`
3. `format` function produces a string by formatting a number of other values according to a specification string. It is similar to the printf function in C, and other similar functions in other programming languages - `format(spec, values...)`
4. `formatlist` produces a list of strings by formatting a number of other values according to a specification string
5. `indent` adds a given number of spaces to the beginnings of all but the first line in a given multi-line string
6. `join` produces a string by concatenating all of the elements of the specified list of strings with the specified separator
7. `lower` converts all cased letters in the given string to lowercase
8. `regex` applies a regular expression to a string and returns the matching substrings. - `regex(pattern, string)`
9. `regexall` applies a regular expression to a string and returns a list of all matches. - `regexall(pattern, string)`
10. `replace` searches a given string for another given substring, and replaces each occurrence with a given replacement string
    `replace(string, substring, replacement)`
11. `split` produces a list by dividing a given string at all occurrences of a given separator - `split(separator, string)`
12. `startswith` takes two values: a string to check and a prefix string. The function returns true if the string begins with that exact prefix - `startswith(string, prefix)`
13. `strcontains` function checks whether a substring is within another string - `strcontains(string, substr)`
14. `strrev` reverses the characters in a string. Note that the characters are treated as Unicode characters - `strrev(string)`
15. `substr` extracts a substring from a given string by offset and (maximum) length - `substr(string, offset, length)`
16. `title` converts the first letter of each word in the given string to uppercase. -  `title("hello world")`
17. `trim` removes the specified set of characters from the start and end of the given string - `trim(string, str_character_set)`
18. `trimprefix` removes the specified prefix from the start of the given string. If the string does not start with the prefix, the string is returned unchanged.  -  `trimprefix("helloworld", "hello")`
19. `trimsuffix` removes the specified suffix from the end of the given string. -  `trimsuffix("helloworld", "world")`
20. `trimspace` removes any space characters from the start and end of the given string
21. `upper` converts all cased letters in the given string to uppercase.

## Collection Functions
1. `alltrue` returns true if all elements in a given collection are true or "true". It also returns true if the collection is empty
2. `anytrue` returns true if any element in a given collection is true or "true". It also returns false if the collection is empty
3. `chunklist` splits a single list into fixed-size chunks, returning a list of lists
4. `coalesce` takes any number of arguments and returns the first one that isn't null or an empty string
5. `coalescelist` takes any number of list arguments and returns the first one that isn't empty
6. `compact` takes a list of strings and returns a new list with any null or empty string elements removed.
7. `concat` takes two or more lists and combines them into a single list
8. `contains` determines whether a given list or set contains a given single value as one of its elements - `contains(list, value)`
9. `distinct` takes a list and returns a new list with any duplicate elements removed. 
10. `element` retrieves a single element from a list. - `element(list, index)`
11. `flatten` takes a list and replaces any elements that are lists with a flattened sequence of the list contents 
   `flatten([["a", "b"], [], ["c"]]) -> ["a", "b", "c"]`
12. `index` finds the element index for a given value in a list - `index(list, value)`
13. `keys` takes a map and returns a list containing the keys from that map - `keys({a=1, c=2, d=3})`
14. `length` determines the length of a given list, map, or string 
15. `lookup` retrieves the value of a single element from a map, given its key. If the given key does not exist, the given default value is returned instead. - `lookup(map, key, default)`
16. `matchkeys` constructs a new list by taking a subset of elements from one list whose indexes match the corresponding indexes of values in another list. - `matchkeys(valueslist, keyslist, searchset)`
17. `merge` takes an arbitrary number of maps or objects, and returns a single map or object that contains a merged set of elements from all arguments
18. `one` takes a list, set, or tuple value with either zero or one elements. If the collection is empty, one returns null. Otherwise, one returns the first element. If there are two or more elements then one will return an error
19. `range` generates a list of numbers using a start value, a limit value, and a step value - `range(max), range(start, limit) , range(start, limit, step)`
20. `reverse` takes a sequence and produces a new sequence of the same length with all of the same elements as the given sequence but in reverse order
21. `setintersection` function takes multiple sets and produces a single set containing only the elements that all of the given sets have in common
22. `setproduct` function finds all of the possible combinations of elements from all of the given sets by computing the Cartesian product.   - `setproduct(sets...)`
23. `setsubtract` function returns a new set containing the elements from the first set that are not present in the second set
24. `setunion` function takes multiple sets and produces a single set containing the elements from all of the given sets
35. `slice` extracts some consecutive elements from within a list - `slice(list, startindex, endindex)`
36. `sort` takes a list of strings and returns a new list with those strings sorted lexicographically
37. `sum` takes a list or set of numbers and returns the sum of those numbers & `sum` fails when given an empty list or set.
38. `transpose` takes a map of lists of strings and swaps the keys and values to produce a new map of lists of strings.
39. `values` takes a map and returns a list containing the values of the elements in that map.
40. `zipmap` constructs a map from a list of keys and a corresponding list of values. - `zipmap(keyslist, valueslist)`

## Encoding Functions
1. `base64decode` takes a string containing a Base64 character sequence and returns the original string.
2. `base64encode` applies Base64 encoding to a string.
3. `base64gzip` compresses a string with gzip and then encodes the result in Base64 encoding.
4. `csvdecode` decodes a string containing CSV-formatted data and produces a list of maps representing that data.
5. `jsondecode` interprets a given string as JSON, returning a representation of the result of decoding that string.
6. `jsonencode` encodes a given value to a string using JSON syntax.
7. `textdecodebase64` function decodes a string that was previously Base64-encoded, and then interprets the result as characters in a specified character encoding.
8. `textencodebase64` encodes the unicode characters in a given string using a specified character encoding, returning the result base64 encoded because Terraform language strings are always sequences of unicode characters
9. `urlencode` applies URL encoding to a given string.
10. `yamldecode` parses a string as a subset of YAML, and produces a representation of its value.
11. `yamlencode` encodes a given value to a string using YAML 1.2 block syntax.

## Filesystem Functions
1. `abspath` takes a string containing a filesystem path and converts it to an absolute path. That is, if the path is not absolute, it will be joined with the current working directory.
2. `dirname` takes a string containing a filesystem path and removes the last portion from it.
3. `pathexpand` takes a filesystem path that might begin with a ~ segment, and if so it replaces that segment with the current user's home directory path.
4. `basename` takes a string containing a filesystem path and removes all except the last portion from it.
5. `file` reads the contents of a file at the given path and returns them as a string
6. `fileexists` determines whether a file exists at a given path
7.  `fileset` enumerates a set of regular file names given a path and pattern. The path is automatically removed from the resulting set of file names and any result still containing path separators always returns forward slash (/) as the path separator for cross-system compatibility. - `fileset(path, pattern)`
8. `filebase64` reads the contents of a file at the given path and returns them as a base64-encoded string. - `filebase64(path)`
9. `templatefile` reads the file at the given path and renders its content as a template using a supplied set of template variables. 
    `templatefile(path, vars)`

## Date & Time Functions
1. `formatdate` converts a timestamp into a different time format. - `formatdate(spec, timestamp)`
```
> formatdate("DD MMM YYYY hh:mm ZZZ", "2018-01-02T23:12:01Z")
02 Jan 2018 23:12 UTC
> formatdate("EEEE, DD-MMM-YY hh:mm:ss ZZZ", "2018-01-02T23:12:01Z")
Tuesday, 02-Jan-18 23:12:01 UTC
> formatdate("EEE, DD MMM YYYY hh:mm:ss ZZZ", "2018-01-02T23:12:01-08:00")
Tue, 02 Jan 2018 23:12:01 -0800
> formatdate("MMM DD, YYYY", "2018-01-02T23:12:01Z")
Jan 02, 2018
> formatdate("HH:mmaa", "2018-01-02T23:12:01Z")
11:12pm
```
2. `plantimestamp` returns a UTC timestamp string in RFC 3339 format. -  `plantimestamp() -> 2018-05-13T07:44:12Z`
3. `timeadd` adds a duration to a timestamp, returning a new timestamp. - `timeadd(timestamp, duration)`
4. `timecmp` compares two timestamps and returns a number that represents the ordering of the instants those timestamps represent.
    `timecmp(timestamp_a, timestamp_b)`
    |Condition	                                    |Return Value     |
    |-----------------------------------------------|-----------------|
    |timestamp_a is before timestamp_b	            |   -1            |
    |timestamp_a is the same instant as timestamp_b	|    0            |
    |timestamp_a is after timestamp_b	            |    1            |
5. `timestamp` returns a UTC timestamp string in RFC 3339 format. -  `timestamp()  - 2018-05-13T07:44:12Z`

## Hash & Crypto Functions
1. `base64sha256` computes the SHA256 hash of a given string and encodes it with Base64. This is not equivalent to `base64encode(sha256("test"))` since sha256() returns hexadecimal representation.
2. `base64sha512` computes the SHA512 hash of a given string and encodes it with Base64. This is not equivalent to `base64encode(sha512("test"))` since sha512() returns hexadecimal representation.
3. `bcrypt` computes a hash of the given string using the Blowfish cipher, returning a string in the Modular Crypt Format usually expected in the shadow password file on many Unix systems. - `bcrypt(string, cost)`,  The `cost` argument is optional and will default to 10 if unspecified.
4. `filebase64sha256` is a variant of base64sha256 that hashes the contents of a given `file` rather than a literal string. This is similar to `base64sha256(file(filename))`, but because file accepts only UTF-8 text it cannot be used to create hashes for binary files.
5. `filebase64sha512` is a variant of base64sha512 that hashes the contents of a given `file` rather than a literal string. This is similar to `base64sha512(file(filename))`, but because file accepts only UTF-8 text it cannot be used to create hashes for binary files.
6. `filemd5` is a variant of md5 that hashes the contents of a given file rather than a literal string. This is similar to `md5(file(filename))`, but because file accepts only UTF-8 text it cannot be used to create hashes for binary files
7. `filesha1` is a variant of sha1 that hashes the contents of a given file rather than a literal string. This is similar to `sha1(file(filename))`, but because file accepts only UTF-8 text it cannot be used to create hashes for binary files
8. `filesha256` is a variant of sha256 that hashes the contents of a given file rather than a literal string. This is similar to `sha256(file(filename))`, but because file accepts only UTF-8 text it cannot be used to create hashes for binary files.
9. `filesha512` is a variant of sha512 that hashes the contents of a given file rather than a literal string. This is similar to `sha512(file(filename))`, but because file accepts only UTF-8 text it cannot be used to create hashes for binary files
10. `md5` computes the MD5 hash of a given string and encodes it with hexadecimal digits.
11. `rsadecrypt` decrypts an RSA-encrypted ciphertext, returning the corresponding cleartext - `rsadecrypt(ciphertext, privatekey)`
12. `sha1` computes the SHA1 hash of a given string and encodes it with hexadecimal digits. The given string is first encoded as UTF-8 and then the SHA1 algorithm is applied
13. `sha256` computes the SHA256 hash of a given string and encodes it with hexadecimal digits. The given string is first encoded as UTF-8 and then the SHA256 algorithm is applied
14. `sha512` computes the SHA512 hash of a given string and encodes it with hexadecimal digits. The given string is first encoded as UTF-8 and then the SHA512 algorithm is applied
15. `uuid` generates UUID-format strings using random bytes. The function generates a well-understood string representation of a 128-bit value, but the output is not RFC-compliant. This function produces a new value each time it is called, and so using it directly in resource arguments will result in spurious diffs. We do not recommend using the uuid function in resource configurations, but it can be used with care in conjunction with the ignore_changes lifecycle meta-argument.
16. `uuidv5` generates a name-based UUID, as described in RFC 4122 section 4.3, also known as a "version 5" UUID
    `uuidv5(namespace, name)` 

## IP Network Functions
1. `cidrhost` calculates a full host IP address for a given host number within a given IP network address prefix.
    `cidrhost(prefix, hostnum)` - `prefix` must be given in CIDR notation, `hostnum` is a whole number that can be represented as a binary integer with no more than the number of digits remaining in the address after the given prefix
    If hostnum is negative, the count starts from the end of the range. 
    For example, cidrhost("10.0.0.0/8", 2) returns 10.0.0.2 and cidrhost("10.0.0.0/8", -2) returns 10.255.255.254
2. `cidrnetmask` converts an IPv4 address prefix given in CIDR notation into a subnet mask address.
    `cidrnetmask(prefix)` -  `prefix` must be given in IPv4 CIDR notation, CIDR notation is the only valid notation for IPv6 addresses, so cidrnetmask produces an error if given an IPv6 address. 
    For Example -  cidrnetmask("172.16.0.0/12") -> 255.240.0.0
3. `cidrsubnet` calculates a subnet address within given IP network address prefix.
    `cidrsubnet(prefix, newbits, netnum)`  
    - `prefix` must be given in CIDR notation, 
    - `newbits` is the number of additional bits with which to extend the prefix. For example, if given a prefix ending in /16 and a newbits value of 4, the resulting subnet address will have length /20.
    - `netnum` is a whole number that can be represented as a binary integer with no more than newbits binary digits, which will be used to populate the additional bits added to the prefix
    - While `cidrhost` allows calculating single host IP addresses, `cidrsubnet` on the other hand creates a new network prefix within the given network prefix
    - `cidrsubnets` can allocate multiple network addresses at once, but numbers them automatically starting with zero.
4. `cidrsubnets` calculates a sequence of consecutive IP address ranges within a particular CIDR prefix.
    `cidrsubnets(prefix, newbits...)` 
    - `cidrsubnet` calculates a single subnet address within a prefix while allowing you to specify its subnet number, while `cidrsubnets` can calculate many at once, potentially of different sizes, and assigns subnet numbers automatically.

## Type Conversion Functions
1. `can` evaluates the given expression and returns a boolean value indicating whether the expression produced a result without any errors. 
    - This is a special function that is able to catch errors produced when evaluating its argument. 
    - The can function can only catch and handle dynamic errors resulting from access to data that isn't known until runtime. It will not catch errors relating to expressions that can be proven to be invalid for any input, such as a malformed resource reference
2. `nonsensitive` takes a sensitive value and returns a copy of that value with the sensitive marking removed, thereby exposing the sensitive value.
    - Normally Terraform tracks when you use expressions to derive a new value from a value that is marked as sensitive, so that the result can also be marked as sensitive.
    - nonsensitive will return an error if you pass a value that isn't marked as sensitive, because such a call would be redundant and potentially confusing or misleading to a future maintainer of your module. Use nonsensitive only after careful consideration and with definite intent
3. `sensitive` takes any value and returns a copy of it marked so that Terraform will treat it as sensitive, with the same meaning and behavior as for sensitive input variables
    - The `sensitive` function might be useful in some less-common situations where a sensitive value arises from a definition within your module, such as if you've loaded sensitive data from a file on disk as part of your configuration
    - ex:
    ```
    locals {
        sensitive_content = sensitive(file("${path.module}/sensitive.txt"))
    }
    ```
4. `tobool` converts its argument to a boolean value.
    - Explicit type conversions are rarely necessary in Terraform because it will convert types automatically where required. Use the explicit type conversion functions only to normalize types returned in module outputs.
    - Only boolean values, null, and the exact strings "true" and "false" can be converted to boolean. All other values will produce an error
5. `tolist` converts its argument to a list value.
    - Pass a set value to tolist to convert it to a list. Since set elements are not ordered, the resulting list will have an undefined order that will be consistent within a particular run of Terraform.
6. `tomap` converts its argument to a map value.
7. `tonumber` converts its argument to a number value
    - Only numbers, null, and strings containing decimal representations of numbers can be converted to number. All other values will produce an error
8. `toset` converts its argument to a set value.'
    - Pass a list value to toset to convert it to a set, which will remove any duplicate elements and discard the ordering of the elements.
9. `tostring` converts its argument to a string value.
    - Only the primitive types (string, number, and bool) and null can be converted to string. tostring(null) produces a null value of type string. All other values produce an error
10. `try` evaluates all of its argument expressions in turn and returns the result of the first one that does not produce any errors.
    - This is a special function that is able to catch errors produced when evaluating its arguments, which is particularly useful when working with complex data structures whose shape is not well-known at implementation time
    ex:
    ```
    locals {
        raw_value = yamldecode(file("${path.module}/example.yaml"))
        normalized_value = {
            name   = tostring(try(local.raw_value.name, null))
            groups = try(local.raw_value.groups, [])
        }
    }
    ```
    -  We strongly suggest using try only in special local values whose expressions perform normalization, so that the error handling is confined to a single location in the module and the rest of the module can just use straightforward references to the normalized structure and thus be more readable for future maintainers
    - The try function can only catch and handle dynamic errors resulting from access to data that isn't known until runtime. It will not catch errors relating to expressions that can be proven to be invalid for any input, such as a malformed resource reference
11. `type` returns the type of a given value.
    - Sometimes a Terraform configuration can result in confusing errors regarding inconsistent types. This function displays terraform's evaluation of a given value's type, which is useful in understanding this error message.
    - This is a special function which is only available in the terraform console command. It can only be used to examine the type of a given value, and should not be used in more complex expressions
