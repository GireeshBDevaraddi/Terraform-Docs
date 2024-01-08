# TERRAFORM STATE
- Terraform uses state to determine which changes to make to your infrastructure. Prior to any operation, Terraform does a refresh to update the state with the real infrastructure.
- While the format of the state files are just JSON, direct file editing of the state is discouraged. Terraform provides the `terraform state `command to perform basic modifications of the state using the CLI.
- what is the Purpose of Terraform State ?
    - Mapping to the Real World
    - Metadata, Alongside the mappings between resources and remote objects, Terraform must also track metadata such as resource dependencies
    - Performance, In addition to basic mapping, Terraform stores a cache of the attribute values for all resources in the state.
    - Syncing

## The `terraform_remote_state` Data Source
- The `terraform_remote_state` data source uses the latest state snapshot from a specified state backend to retrieve the root module output values from some other Terraform configuration
- You can use the `terraform_remote_state` data source without requiring or configuring a provider. It is always available through a built-in provider with the source address `terraform.io/builtin/terraform`. That provider does not include any other resources or data sources
- Sharing data with root module outputs is convenient, but it has drawbacks. Although `terraform_remote_state` only exposes output values, its user must have access to the entire state snapshot, which often includes some sensitive information. When possible, we recommend explicitly publishing data for external consumption to a separate location instead of accessing it via remote state. This lets you apply different access controls for shared information and state snapshots.
- To share data explicitly between configurations, you can use pairs of managed resource types and data sources in various providers.

## State Storage and Locking
- Backends are responsible for storing state and providing an API for state locking. State locking is optional
- In the case of an error persisting the state to the backend, Terraform will write the state locally. This is to prevent data loss. If this happens, the end user must manually push the state to the remote backend once the error is resolved.
- You can still manually retrieve the state from the remote state using the `terraform state pull` command. This will load your remote state and output it to stdout. You can choose to save that to a file or perform any other operations
- You can also manually write state with `terraform state push`. This is extremely dangerous and should be avoided if possible. This will overwrite the remote state. This can be used to do manual fixups if necessary.
- Terraform has a `force-unlock` command to manually unlock the state if unlocking failed.
- Be very careful with this command. If you unlock the state when someone else is holding the lock it could cause multiple writers. Force unlock should only be used to unlock your own lock in the situation where automatic unlocking failed.
- To protect you, the `force-unlock` command requires a unique lock ID. Terraform will output this lock ID if unlocking fails. This lock ID acts as a nonce, ensuring that locks and unlocks target the correct lock.

## workspaces
- Each Terraform configuration has an associated backend that defines how Terraform executes operations and where Terraform stores persistent data, like state
- The persistent data stored in the backend belongs to a workspace. The backend initially has only one workspace containing one Terraform state associated with that configuration. Some backends support multiple named workspaces, allowing multiple states to be associated with a single configuration. The configuration still has only one backend, but you can deploy multiple distinct instances of that configuration without configuring a new backend or changing authentication credentials
- Terraform starts with a single, default workspace named `default` that you cannot delete. If you have not created a new workspace, you are using the `default` workspace in your Terraform working directory.
- When you run `terraform plan` in a new workspace, Terraform does not access existing resources in other workspaces. These resources still physically exist, but you must switch workspaces to manage them
- Within your Terraform configuration, you may include the name of the current workspace using the `${terraform.workspace}` interpolation sequence. This can be used anywhere interpolations are allowed.
