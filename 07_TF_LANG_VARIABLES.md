# VARIABLES & OUTPUTS
## Input Variables 
- serve as parameters for a Terraform module, so users can customize behavior without editing the source.
- Input variables let you customize aspects of Terraform modules without altering the module's own source code. This functionality allows you to share modules across different Terraform configurations, making your module composable and reusable
- Input variables are like function arguments
- For brevity, input variables are often referred to as just "variables" or "Terraform variables" when it is clear from context what sort of variable is being discussed. Other kinds of variables in Terraform include environment variables (set by the shell where Terraform runs) and expression variables (used to indirectly represent a value in an expression).
- The label after the `variable` keyword is a name for the variable, which must be unique among all variables in the same module
  ```
    variable "image_id" {
      type = string
    }
  ```
- The name of a variable can be any valid identifier except these: `source`, `version`, `providers`, `count`, `for_each`, `lifecycle`, `depends_on`, `locals`
- Terraform CLI defines the following optional arguments for variable declarations:
  - default       : A default value that then makes the variable optional.
  - type          : This argument specifies what value types are accepted for the variable.
    - string
    - number
    - bool
    - list(<TYPE>)
    - set(<TYPE>)
    - map(<TYPE>)
    - object({<ATTR NAME> = <TYPE>, ... })
    - tuple([<TYPE>, ...])
    - The keyword 'any' may be used to indicate that any type is acceptable
  - description   : This specifies the input variable's documentation.
  - validation    : A block to define validation rules, usually in addition to type constraints.
    - You can specify custom validation rules for a particular variable by adding a validation block within the corresponding variable block
  - sensitive     : Limits Terraform UI output when the variable is used in configuration.
  - nullable      : Specify if the variable can be null within the module. The default value for nullable is true
- If both the type and default arguments are specified, the given default value must be convertible to the specified type
- Within the module that declared a variable, its value can be accessed from within expressions as var.<NAME>, where <NAME> matches the label given in the declaration block ex: var.image_id
- When variables are declared in the root module of your configuration, they can be set in a number of ways:
  1. In a Terraform Cloud workspace.
  2. Individually, with the `-var` command line option.
  3. In variable definitions (.tfvars or .tfvars.json) files, either specified on the command line or automatically loaded. (`-var-file`)
    - Terraform also automatically loads a number of variable definitions files if they are present:
      * Files named exactly terraform.tfvars or terraform.tfvars.json.
      * Any files with names ending in .auto.tfvars or .auto.tfvars.json.
  4. As environment variables 
    - As a fallback for the other ways of defining variables, Terraform searches the environment of its own process for environment variables named TF_VAR_ followed by the name of a declared variable
- The above mechanisms for setting variables can be used together in any combination. If the same variable is assigned multiple values, Terraform uses the last value it finds, overriding any previous values. Note that the same variable cannot be assigned multiple values within a single source
- Terraform loads variables in the following order, with later sources taking precedence over earlier ones:
  1. Environment variables
  2. The terraform.tfvars file, if present.
  3. The terraform.tfvars.json file, if present.
  4. Any *.auto.tfvars or *.auto.tfvars.json files, processed in lexical order of their filenames.
  5. Any -var and -var-file options on the command line, in the order they are provided. (This includes variables set by a Terraform Cloud workspace.)
- Within Terraform test files, you can specify variable values within variable blocks, either nested within run blocks or defined directly within the file. Variables defined in this way take precedence over all other mechanisms during test execution, with variables defined within run blocks taking precedence over those defined within the file

## Output Variables   
- Output values are like function return values or return values for a Terraform module.
- Output values make information about your infrastructure available on the command line, and can expose information for other Terraform configurations to use. Output values are similar to return values in programming languages.
- Output values have several uses:
  1. A child module can use outputs to expose a subset of its resource attributes to a parent module.
  2. A root module can use outputs to print certain values in the CLI output after running `terraform apply`.
  3. When using a remote state, root module outputs can be accessed by other configurations via a terraform_remote_state data source.
- Resource instances managed by Terraform each export attribute whose values can be used elsewhere in configuration. Output values are a way to expose some of that information to the user of your module
- Each output value exported by a module must be declared using an `output` block
    ```
    output "instance_ip_addr" {
      value = aws_instance.server.private_ip
    }
    ```
- The `value` argument takes an expression whose result is to be returned to the user.
- Outputs are only rendered when Terraform applies your plan. Running `terraform plan` will not render outputs
- In a parent module, outputs of child modules are available in expressions as `module.<MODULE NAME>.<OUTPUT NAME>`
- You can use `precondition` blocks to specify guarantees about output data
- `output` blocks can optionally include `description`, `sensitive`, and `depends_on` arguments

## Local Variables  
- Local values are like a function's temporary local variables, a convenience feature for assigning a short name to an expression.
- A local value assigns a name to an expression, so you can use the name multiple times within a module instead of repeating the expression
- A set of related local values can be declared together in a single locals block
```
locals {
  service_name = "forum"
  owner        = "Community Team"
}
```
- The expressions in local values are not limited to literal constants; they can also reference other values in the module in order to transform or combine them, including variables, resource attributes, or other local values
```
locals {
  # Ids for multiple sets of EC2 instances, merged together
  instance_ids = concat(aws_instance.blue.*.id, aws_instance.green.*.id)
}
```
- Once a local value is declared, you can reference it in expressions as `local.<NAME>`
- Local values can be helpful to avoid repeating the same values or expressions multiple times in a configuration, but if overused they can also make a configuration hard to read by future maintainers by hiding the actual values used
