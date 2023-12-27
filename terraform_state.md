Source - https://developer.hashicorp.com/terraform/tutorials/state

# Import Terraform configuration
- Terraform stores information about your infrastructure in a state file. To update your infrastructure, you first modify your configuration, and then use Terraform to plan and apply the required changes. Terraform uses the data in your state file to determine the changes it needs to make to your infrastructure.
- Use terraform plan with the -generate-config-out flag to generate configuration for the container you will import. Terraform builds a plan and outputs the generated configuration for the container to the specified file.

# Migrate state to Terraform Cloud
- Terrafrom Cloud offers secure remote state storage, make it easier to collaborate on infrastructure development. You can migrate your state to Terraform Cloud without interrupting services or recreating your existing infrastructure.
- you need to create a cloud code block in your configuration. To use Terraform Cloud as a backend for your configuration, you must include a cloud block in your configuration. Add the cloud block to your configuration as shown below, replacing ORGANIZATION-NAME with your own Terraform Cloud organization name.
- While the organization defined in the cloud stanza must already exist, the workspace does not have to; Terraform Cloud will create it if necessary. If you use an existing workspace, it must not have any existing states.
- After configuring your Terraform Cloud integration, you must authenticate to Terraform Cloud to use it for remote operations.
    Run terraform login and follow the prompts to log in, typing yes at the confirmation prompt.
    
# Manage resources in Terraform state
- Terraform stores information about your infrastructure in a state file. This state file keeps track of resources created by your configuration and maps them to real-world resources.
- Terraform compares your configuration with the state file and your existing infrastructure to create plans and make changes to your infrastructure. When you run terraform apply or terraform destroy against your initialized configuration, Terraform writes metadata about your configuration to the state file and updates your infrastructure resources accordingly.
- Run terraform state list to get the list of resource names and local identifiers in your state file. This command is useful for more complex configurations where you need to find a specific resource without parsing state with terraform show.
- Terraform usually only updates your infrastructure if it does not match your configuration. You can use the -replace flag for terraform plan and terraform apply operations to safely recreate resources in your environment even if you have not edited the configuration, which can be useful in cases of system malfunction
- you may have used the terraform taint command to achieve a similar outcome. That command has now been deprecated in favor of the -replace flag, which allows for a simpler, less error-prone workflow.
- The terraform state mv command moves resources from one state file to another. You can also rename resources with mv. The move command will update the resource in state, but not in your configuration file. Moving resources is useful when you want to combine modules or resources from other states, but do not want to destroy and recreate the infrastructure.
	- The terraform state rm subcommand removes specific resources from your state file. This does not remove the resource from your configuration or destroy the infrastructure itself.
    - Run terraform import to bring this security group back into your state file. Removing the security group from state did not remove the output value with its ID, so you can use it for the import.
    - The terraform refresh command updates the state file when physical resources change outside of the Terraform workflow.

# Target resources
- You can use Terraform's -target option to target specific resources, modules, or collections of resources
- When using Terraform's normal workflow and applying changes to the entire working directory, the bucket name modification would apply to all downstream dependencies as well. Because you targeted the random pet resource, Terraform updated the output value for the bucket name but not the bucket itself. Targeting resources can introduce inconsistencies, so you should only use it in troubleshooting scenarios.

# Troubleshoot Terraform

# Manage resource drift
- If you suspect that your infrastructure configuration changed outside of the Terraform workflow, you can use a -refresh-only flag to inspect what the changes to your state file would be. This is safer than the refresh subcommand, which automatically overwrites your state file without displaying the updates
- Run terraform plan -refresh-only to determine the drift between your current state file and actual configuration.
- use terraform import for the respective changed resource to match the state file configuration

# Manage Resource lifecycle
- Lifecycle arguments help control the flow of your Terraform operations by creating custom rules for resource creation and destruction. Instead of Terraform managing operations in the built-in dependency graph, lifecycle arguments help minimize potential downtime based on your resource needs as well as protect specific resources from changing or impacting infrastructure
- To prevent destroy operations for specific resources, you can add the prevent_destroy attribute to your resource definition. This lifecycle option prevents Terraform from accidentally removing critical resources
- For changes that may cause downtime but must happen, use the create_before_destroy attribute to create your new resource before destroying the old resource.
- For changes outside the Terraform workflow that should not impact Terraform operations, use the ignore_changes argument 
  - prevent_destroy
  - create_before_destroy
  - ignore_changes
  ex: Add the lifecyle block to the resource block.
    lifecycle {
      create_before_destroy = true
      ignore_changes        = [tags]
    }
# Version remote state with the Terraform Cloud API
# Refresh State
# console
# Use configuration to move resources
- The moved configuration block lets you track your resource moves in the configuration itself. With the moved configuration block, you can plan, preview, and validate resource moves, enabling you to safely refactor your configuration.
