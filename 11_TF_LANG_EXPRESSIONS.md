# EXPRESSIONS
- `Expressions` refer to or compute values within a configuration. The simplest expressions are just literal values, like "hello" or 5, but the Terraform language also allows more complex expressions such as references to data exported by resources, arithmetic, conditional evaluation, and a number of built-in functions
- You can experiment with the behavior of Terraform's expressions from the Terraform expression console, by running the `terraform console` command.
## Types and Values
- The result of an expression is a value. All values have a type, which dictates where that value can be used and what transformations can be applied to it
- The Terraform language uses the following types for its values:
    1. `string` : a sequence of Unicode characters representing some text, like `"hello"`.
    2. `number` : a numeric value. The number type can represent both whole numbers like 15 and fractional values like `6.283185`.
    3. `bool` : a boolean value, either `true` or `false`. bool values can be used in conditional logic.
    4. `list` (or `tuple`) : a sequence of values, like `["us-west-1a", "us-west-1c"]`. Identify elements in a list with consecutive whole numbers, starting with zero.
    5. `set`: a collection of unique values that do not have any secondary identifiers or ordering.
    6. `map` (or `object`) : a group of values identified by named labels, like `{name = "Mabel", age = 52}`. The keys in a map must be strings, You can use a non-literal string expression as a key by wrapping it in parentheses, like `(var.business_unit_tag_name) = "SRE"`.
- Strings, numbers, and bools are sometimes called `primitive types`. Lists/tuples and maps/objects are sometimes called `complex types, structural types, or collection types`
- `null` : a value that represents absence or omission. If you set an argument of a resource to null, Terraform behaves as though you had completely omitted it — it will use the argument's default value if it has one, or raise an error if the argument is mandatory. null is most useful in conditional expressions, so you can dynamically omit an argument if a condition isn't met.
- Terraform automatically converts values from one type to another in order to produce the expected type. If this isn't possible, Terraform will produce a type mismatch error and you must update the configuration with a more suitable expression
## Strings and Templates
- String literals are the most complex kind of literal expression in Terraform, and also the most commonly used. Terraform supports both a quoted syntax and a "heredoc" syntax for strings. Both of these syntaxes support template sequences for interpolating values and manipulating text.
```
"hello"
# Heredoc
<<EOT
hello
world
EOT
```
- Don't use `"heredoc"` strings to generate JSON or YAML. Instead, use the `jsonencode` function or the `yamlencode` function so that Terraform can be responsible for guaranteeing valid JSON or YAML syntax
- A ${ ... } sequence is an interpolation, which evaluates the expression given between the markers, converts the result to a string if necessary, and then inserts it into the final string 
- A %{ ... } sequence is a directive, which allows for conditional results and iteration over collections, similar to conditional and for expressions
- The `%{if <BOOL>}/%{else}/%{endif}` directive chooses between two templates based on the value of a bool expression
```
"Hello, %{ if var.name != "" }${var.name}%{ else }unnamed%{ endif }!"
```
- The `%{for <NAME> in <COLLECTION>} / %{endfor}` directive iterates over the elements of a given collection or structural value and evaluates a given template once for each element, concatenating the results together
```
<<EOT
%{ for ip in aws_instance.example[*].private_ip }
server ${ip}
%{ endfor }
EOT
```
- To allow template directives to be formatted for readability without adding unwanted spaces and newlines to the result, all template sequences can include optional strip markers (~)
```
<<EOT
%{ for ip in aws_instance.example[*].private_ip ~}
server ${ip}
%{ endfor ~}
EOT
```
- When using template directives, we recommend always using the "heredoc" string literal form and then formatting the template over multiple lines for readability. Quoted string literals should usually include only interpolation sequences.
## References to Values
- Terraform makes several kinds of named values available. Each of these names is an expression that references the associated value. You can use them as standalone expressions, or combine them with other expressions to compute new values.
- The main kinds of named values available in Terraform are:
    1. Resources                        - <RESOURCE TYPE>.<NAME>
    2. Input variables                  - var.<NAME> 
    3. Local values                     - local.<NAME> 
    4. Child module outputs             - module.<MODULE NAME>
    5. Data sources                     - data.<DATA TYPE>.<NAME>
    6. Filesystem and workspace info    - path.module, path.root, path.cwd, terraform.workspace
    7. Block-local values               - count.index, each.key / each.value, self
## Operators
- An operator is a type of expression that transforms or combines one or more other expressions. Operators either combine two values in some way to produce a third result value, or transform a single given value to produce a single result
- When multiple operators are used together in an expression, they are evaluated in the following order of operations
    1. !, - (multiplication by -1)
    2. *, /, %
    3. +, - (subtraction)
    4. >, >=, <, <=
    5. ==, !=
    6. &&
    7. ||
- Use parentheses to override the default order of operations. Without parentheses, higher levels will be evaluated first
- Arithmetic Operators
- Equality Operators
- Comparison Operators
- Logical Operators
## Function Calls
- The Terraform language has a number of built-in functions that can be used in expressions to transform and combine values. These are similar to the operators but all follow a common syntax
```
<FUNCTION NAME>(<ARGUMENT 1>, <ARGUMENT 2>, ...)
min(55, 3453, 2)
```
- If the arguments to pass to a function are available in a list or tuple value, that value can be expanded into separate arguments. Provide the list value as an argument and follow it with the ... symbol
## Conditional Expressions
- A conditional expression uses the value of a boolean expression to select one of two values. 
`condition ? true_val : false_val`
- The `condition` can be any expression that resolves to a boolean value. This will usually be an expression that uses the equality, comparison, or logical operators
## For Expressions
- A `for` expression creates a complex type value by transforming another complex type value. Each element in the input value can correspond to either one or zero values in the result, and an arbitrary expression can be used to transform each input element into an output element
`[for s in var.list : upper(s)]`
- A `for` expression's input (given after the in keyword) can be a list, a set, a tuple, a map, or an object. The above example showed a for expression with only a single temporary symbol s, but a for expression can optionally declare a pair of temporary symbols in order to use the key or index of each item too:
`[for k, v in var.map : length(k) + length(v)]`
- A `for` expression can also include an optional if clause to filter elements from the source collection, producing a value with fewer elements than the source value
`[for s in var.list : upper(s) if s != ""]`
- for expressions can convert from unordered types (maps, objects, sets) to ordered types (lists, tuples), Terraform must choose an implied ordering for the elements of an unordered collection.
- For maps and objects, Terraform sorts the elements by key or attribute name, using lexical sorting
- For sets of strings, Terraform sorts the elements by their value, using lexical sorting.
- For sets of other types, Terraform uses an arbitrary ordering that may change in future versions. We recommend converting the expression result into a set to make it clear elsewhere in the configuration that the result is unordered. You can use the toset function to concisely convert a for expression result to be of a set type.
`toset([for e in var.set : e.example])`
- To activate grouping mode, add the symbol ... after the value expression
```
variable "users" {
  type = map(object({
    role = string
  }))
}

locals {
  users_by_role = {
    for name, user in var.users : user.role => name...
  }
}
```
## Splat Expressions
- A splat expression provides a more concise way to express a common operation that could otherwise be performed with a for expression
- `[for o in var.list : o.id]` is equivalent to splat expression `var.list[*].id`
- The splat expression patterns shown above apply only to `lists`, `sets`, and `tuples`. To get a similar result with a `map` or `object` value you must use `for` expressions. Resources that use the `for_each` argument will appear in expressions as a map of objects, so you can't use splat expressions with those resources
## Dynamic Blocks
- Within top-level block constructs like resources, expressions can usually be used only when assigning a value to an argument using the name = expression form. This covers many uses, but some resource types include repeatable nested blocks in their arguments, which typically represent separate objects that are related to (or embedded within) the containing object
```
resource "aws_elastic_beanstalk_environment" "tfenvtest" {
  name = "tf-test-name" # can use expressions here
  application         = "${aws_elastic_beanstalk_application.tftest.name}"
  solution_stack_name = "64bit Amazon Linux 2018.03 v2.11.4 running Go 1.12.6"

  setting {
    # but the "setting" block is always a literal block
  }
}
```
- You can dynamically construct repeatable nested blocks like setting using a special `dynamic` block type, which is supported inside `resource`, `data`, `provider`, and `provisioner` blocks
```
resource "aws_elastic_beanstalk_environment" "tfenvtest" {
  name                = "tf-test-name"
  application         = "${aws_elastic_beanstalk_application.tftest.name}"
  solution_stack_name = "64bit Amazon Linux 2018.03 v2.11.4 running Go 1.12.6"

  dynamic "setting" {
    for_each = var.settings
    content {
      namespace = setting.value["namespace"]
      name = setting.value["name"]
      value = setting.value["value"]
    }
  }
}
```
- A `dynamic` block acts much like a for expression, but produces nested blocks instead of a complex typed value. It iterates over a given complex value, and generates a nested block for each element of that complex value
    - The label of the `dynamic` block (`"setting"` in the example above) specifies what kind of nested block to generate.
    - The `for_each` argument provides the complex value to iterate over.
    - The `iterator` argument (optional) sets the name of a temporary variable that represents the current element of the complex value. If omitted, the name of the variable defaults to the label of the dynamic block ("setting" in the example above).
    - The `labels` argument (optional) is a list of strings that specifies the block labels, in order, to use for each generated block. You can use the temporary iterator variable in this value.
    - The nested `content` block defines the body of each generated block. You can use the temporary iterator variable inside this block.
- The iterator object (setting in the example above) has two attributes:
    - `key` is the map key or list element index for the current element. If the for_each expression produces a set value then key is identical to value and should not be used.
    - `value` is the value of the current element
## Custom Conditions
- Custom conditions can capture assumptions, helping future maintainers understand the configuration design and intent. They also return useful information about errors earlier and in context, helping consumers more easily diagnose issues in their configurations
- When Terraform evaluates custom conditions during the plan and apply cycle
- Use the following broad guidelines to select the best custom condition for your use case
    1. `Check blocks with assertions` validate your infrastructure as a whole.
    2. `Validation conditions` or `output postconditions` can ensure your configuration's inputs and outputs meet specific requirements
    3. Resource `preconditions` and `postconditions` can validate that Terraform produces your configuration with predictable results.
- Add one or more `validation` blocks within the `variable` block to specify custom conditions. Each validation requires a `condition` argument for evaluation and `error_message` argument to show the message for false evaluations.
- The `lifecycle` block inside a `resource` or `data` block can include both `precondition` and `postcondition` blocks.
- An `output` block can include a `precondition` block. Preconditions can serve a symmetrical purpose to input variable validation blocks.
- Use Preconditions for Assumptions and  Use Postconditions for Guarantees
## Type Constraints
- Terraform module authors and provider developers can use detailed type constraints to validate user-provided values for their input variables and resource arguments. This requires some additional knowledge about Terraform's type system, but allows you to build a more resilient user interface for your modules and resources.
- Type constraints are expressed using a mixture of type keywords and function-like constructs called type constructors
- The keyword `any` is a special construct that serves as a placeholder for a type yet to be decided. any is not itself a type: when interpreting a value against a type constraint containing any, Terraform will attempt to find a single actual type that could replace the any keyword to produce a valid result
- Terraform typically returns an error when it does not receive a value for specified object attributes. When you mark an attribute as optional, Terraform instead inserts a default value for the missing attribute. This allows the receiving module to describe an appropriate fallback behavior.
- To mark attributes as optional, use the optional modifier in the object type constraint
```
variable "with_optional_attribute" {
  type = object({
    a = string                # a required attribute
    b = optional(string)      # an optional attribute
    c = optional(number, 127) # an optional attribute with default value
  })
}
```
- The optional modifier takes one or two arguments.
    1. Type: (Required) The first argument specifies the type of the attribute.
    2. Default: (Optional) The second argument defines the default value that Terraform should use if the attribute is not present. This must be compatible with the attribute type. If not specified, Terraform uses a null value of the appropriate type as the default.
## Version Constraints
- Anywhere that Terraform lets you specify a range of acceptable versions for something, it expects a specially formatted string known as a version constraint. Version constraints are used when configuring:
    1. Modules
    2. Provider requirements
    3. The required_version setting in the terraform block
- `version = ">= 1.2.0, < 2.0.0"`
- The following operators are valid:
    1. = (or no operator): Allows only one exact version number. Cannot be combined with other conditions.
    2. !=: Excludes an exact version number.
    3. >, >=, <, <=: Comparisons against a specified version, allowing versions for which the comparison is true. "Greater-than" requests newer versions, and "less-than" requests older versions.
    4. ~>: Allows only the rightmost version component to increment. This format is referred to as the pessimistic constraint operator. For example, to allow new patch releases within a specific minor release, use the full version number:
        ~> 1.0.4: Allows Terraform to install 1.0.5 and 1.0.10 but not 1.1.0.
        ~> 1.1: Allows Terraform to install 1.2 and 1.10 but not 2.0.
