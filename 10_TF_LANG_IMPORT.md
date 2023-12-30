# IMPORT
- Use the `import` block to import existing infrastructure resources into Terraform, bringing them under Terraform's management. Unlike the `terraform import` command, configuration-driven import using `import` blocks is predictable, works with CICD pipelines, and lets you preview an import operation before modifying state
- The `import` block records that Terraform imported the resource and did not create it. After importing, you can optionally remove import blocks from your configuration or leave them as a record of the resource's origin
## Syntax
- You can add an `import` block to any Terraform configuration file. A common pattern is to create an `imports.tf` file, or to place each `import` block beside the `resource` block it imports into
```
import {
  to = aws_instance.example
  id = "i-abcd1234"
}

resource "aws_instance" "example" {
  name = "hashi"
  # (other resource arguments...)
}
```
-  The import block has the following arguments
    1. to - The instance address this resource will have in your state file.
    2. id - A string with the import ID of the resource.
    3. provider (optional) - An optional custom resource provider,
- The import block's `id` argument can be a literal string of your resource's import ID, or an expression that evaluates to a string. Terraform needs this import ID to locate the resource you want to import
- The import ID must be known at plan time for planning to succeed. If the value of id is only known after apply, `terraform plan` will fail with an error.
- The `import` block is `idempotent`, meaning that applying an import action and running another plan will not generate another import action as long as that resource remains in your state, You can remove `import` blocks after completing the import or safely leave them in your configuration as a record of the resource's origin for future module maintainers
## Generating configuration
- Terraform can generate code for the resources you define in `import` blocks that do not already exist in your configuration. Terraform produces HCL to act as a template that contains Terraform's best guess at the appropriate value for each resource argument.
- To generate configuration, run `terraform plan` with the `-generate-config-out` flag and supply a new file path. Do not supply a path to an existing file, or Terraform throws an error
```
$terraform plan -generate-config-out=generated_resources.tf
```
