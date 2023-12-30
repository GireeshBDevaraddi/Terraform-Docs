# SYNTAX 
- The majority of the Terraform language documentation focuses on the practical uses of the language and the specific constructs it uses
## Configuration Syntax
- describes the native grammar of the Terraform language.
- This describes the lower-level syntax of the language in more detail, revealing the building blocks that those constructs are built from.
- This describes the native syntax of the Terraform language, which is a rich language designed to be relatively easy for humans to read and write. The constructs in the Terraform language can also be expressed in JSON syntax, which is harder for humans to read and edit but easier to generate and parse programmatically.
- This low-level syntax of the Terraform language is defined in terms of a syntax called HCL, which is also used by configuration languages in other applications, and in particular other HashiCorp products. It is not necessary to know all of the details of HCL syntax in order to use Terraform
- The Terraform language syntax is built around two key syntax constructs: arguments and blocks
  - arguments
    - An argument assigns a value to a particular name
      ex: image_id = "abc123"
    - The identifier before the equals sign is the argument name, and the expression after the equals sign is the argument's value
    - Terraform's configuration language is based on a more general language called HCL, and HCL's documentation usually uses the word "attribute" instead of "argument." These words are similar enough to be interchangeable in this context, and experienced Terraform users might use either term in casual conversation
  - blocks.
    - A block is a container for other content
    ex:
        ```
        resource "aws_instance" "example" {
          ami = "abc123"
          network_interface {
            # ...
          }
        }
        ```
    - A block has a type (resource in this example). Each block type defines how many labels must follow the type keyword. The resource block type expects two labels, which are aws_instance and example in the example above. A particular block type may have any number of required labels, or it may require none as with the nested network_interface block type.
    - After the block type keyword and any labels, the block body is delimited by the { and } characters. Within the block body, further arguments and blocks may be nested, creating a hierarchy of blocks and their associated arguments.
    - The Terraform language uses a limited number of top-level block types, which are blocks that can appear outside of any other block in a configuration file. Most of Terraform's features (including resources, input variables, output values, data sources, etc.) are implemented as top-level blocks.
  - Identifiers
    -  Argument names, block type names, and the names of most Terraform-specific constructs like resources, input variables, etc. are all identifiers.
    - Identifiers can contain letters, digits, underscores (_), and hyphens (-). The first character of an identifier must not be a digit, to avoid ambiguity with literal numbers.
  - Comments
    - The Terraform language supports three different syntaxes for comments:
    - \# begins a single-line comment, ending at the end of the line.
    - // also begins a single-line comment, as an alternative to #.
    - /* and */ are start and end delimiters for a comment that might span over multiple lines.
    - The # single-line comment style is the default comment style and should be used in most cases. Automatic configuration formatting tools may automatically transform // comments into # comments, since the double-slash style is not idiomatic.
    - Terraform configuration files must always be UTF-8 encoded. While the delimiters of the language are all ASCII characters, Terraform accepts non-ASCII characters in identifiers, comments, and string values.
    - Terraform accepts configuration files with either Unix-style line endings (LF only) or Windows-style line endings (CR then LF), but the idiomatic style is to use the Unix convention, and so automatic configuration formatting tools may automatically transform CRLF endings to LF.

## JSON Configuration Syntax 
- documents how to represent Terraform language constructs in the pure JSON variant of the Terraform language. Terraform's JSON syntax is unfriendly to humans, but can be very useful when generating infrastructure as code with other systems that don't have a readily available HCL library.
- The JSON syntax is defined in terms of the native syntax. Everything that can be expressed in native syntax can also be expressed in JSON syntax, but some constructs are more complex to represent in JSON due to limitations of the JSON grammar.
- At the root of any JSON-based Terraform configuration is a JSON object. The properties of this object correspond to the top-level block types of the Terraform language
        Ex:
        ```
        {
          "resource": {
            "aws_instance": {
              "example": {
                "instance_type": "t2.micro",
                "ami": "ami-abc123"
              }
            }
          }
        }
        ```
- When a JSON string is encountered in a location where arbitrary expressions are expected, its value is first parsed as a string template and then it is evaluated to produce the final result
- If a nested block type requires labels but the order does not matter, you may omit the array and provide just a single object whose property names correspond to unique block labels.
- In any object that represents a block body, properties named "//" are ignored by Terraform entirely. This exception does not apply to objects that are being interpreted as expressions, where this would be interpreted as an object type attribute named "//".
        ```
          {
            "resource": {
              "aws_instance": {
                "example": {
                  "//": "This instance runs the scheduled tasks for backup",
                  "instance_type": "t2.micro",
                  "ami": "ami-abc123"
                }
              }
            }
          }
        ```
- Some meta-arguments for the resource and data block types take direct references to objects, or literal keywords. When represented in JSON, the reference or keyword is given as a JSON string with no additional surrounding spaces or symbols.
- Special processing also applies to the type argument of any connection blocks, whether directly inside the resource block or nested inside provisioner blocks: the given string is interpreted literally, and not parsed and evaluated as a string template.
- 
## Style Conventions
- documents some commonly accepted formatting guidelines for Terraform code. These conventions can be enforced automatically with terraform fmt.
- Indent two spaces for each nesting level
- Use empty lines to separate logical groups of arguments within a block
- For blocks that contain both arguments and "meta-arguments" (as defined by the Terraform language semantics), list meta-arguments first and separate them from other arguments with one blank line. Place meta-argument blocks last and separate them from other blocks with one blank line.
- Avoid grouping multiple blocks of the same type with other blocks of a different type, unless the block types are defined by semantics to form a family.
