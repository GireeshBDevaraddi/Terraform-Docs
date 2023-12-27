# RESOURCES 
- Resources are the most important element in the Terraform language. Each resource block describes one or more infrastructure objects, such as virtual networks, compute instances, or higher-level components such as DNS records.
  - Resource Blocks documents the syntax for declaring resources.
  - Resource Behavior explains in more detail how Terraform handles resource declarations when applying a configuration.
  - The Meta-Arguments section documents special arguments that can be used with every resource type, including depends_on, count, for_each, provider, and lifecycle.
  - Provisioners documents configuring post-creation actions for a resource using the provisioner and connection blocks. Since provisioners are non-declarative and potentially unpredictable, we strongly recommend that you treat them as a last resort.
  - Resource names must start with a letter or underscore, and may contain only letters, digits, underscores, and dashes.
  - Resource declarations can include more advanced features, such as single resource declarations that produce multiple similar remote objects, but only a small subset is required for initial use
  - Each resource is associated with a single resource type, which determines the kind of infrastructure object it manages and what arguments and other attributes the resource supports
  - You can use precondition and postcondition blocks to specify assumptions and guarantees about how the resource operates.
  - Custom Condition Checks can help capture assumptions, helping future maintainers understand the configuration design and intent. They also return useful information about errors earlier and in context, helping consumers to diagnose issues in their configuration.
  - Some resource types provide a special timeouts nested block argument that allows you to customize how long certain operations are allowed to take before being considered to have failed
  - Timeouts are handled entirely by the resource type implementation in the provider, but resource types offering these features follow the convention of defining a child block called timeouts that has a nested argument named after each operation that has a configurable timeout value. Each of these arguments takes a string representation of a duration.
  - The set of configurable operations is chosen by each resource type. Most resource types do not support the timeouts block at all.

  - Applying a Terraform configuration will:
    - Create resources that exist in the configuration but are not associated with a real infrastructure object in the state.
    - Destroy resources that exist in the state but no longer exist in the configuration.
    - Update in-place resources whose arguments have changed.
    - Destroy and re-create resources whose arguments have changed but which cannot be updated in-place due to remote API limitations.
  - We can also use the replace_triggered_by meta-argument to add dependencies between otherwise independent resources. It forces Terraform to replace the parent resource when there is a change to a referenced resource or resource attribute.
  - While most resource types correspond to an infrastructure object type that is managed via a remote network API, there are certain specialized resource types that operate only within Terraform itself, calculating some results and saving those results in the state for future use. 
  For example, local-only resource types exist for generating private keys, issuing self-signed TLS certificates, and even generating random ids. 
  - The behavior of local-only resources is the same as all other resources, but their result data exists only within the Terraform state. "Destroying" such a resource means only to remove it from the state, discarding its data.

## Meta-Arguments
- depends_on
  - Use the depends_on meta-argument to handle hidden resource or module dependencies that Terraform cannot automatically infer. You only need to explicitly specify a dependency when a resource or module relies on another resource's behavior but does not access any of that resource's data in its arguments.
  - Module support for depends_on was added in Terraform version 0.13, and prior versions can only use it with resources.
  - The depends_on meta-argument instructs Terraform to complete all actions on the dependency object (including Read actions) before performing actions on the object declaring the dependency. When the dependency object is an entire module, depends_on affects the order in which Terraform processes all of the resources and data sources associated with that module
  - You should use depends_on as a last resort because it can cause Terraform to create more conservative plans that replace more resources than necessary
  - Instead of depends_on, we recommend using expression references to imply dependencies when possible. Expression references let Terraform understand which value the reference derives from and avoid planning changes if that particular value hasn’t changed, even if other parts of the upstream object have planned changes.
  - We recommend always including a comment that explains why using depends_on is necessary.
    ```
    depends_on = [
                aws_iam_role_policy.example
            ]
    ```
- Count
  - Module support for count was added in Terraform 0.13, and previous versions can only use it with resources
  - By default, a resource block configures one real infrastructure object. (Similarly, a module block includes a child module's contents into the configuration one time.) However, sometimes you want to manage several similar objects (like a fixed pool of compute instances) without writing a separate block for each one. Terraform has two ways to do this: count and for_each.
  - If a resource or module block includes a count argument whose value is a whole number, Terraform will create that many instances.
  - count is a meta-argument defined by the Terraform language. It can be used with modules and with every resource type.
  -  Each instance has a distinct infrastructure object associated with it, and each is separately created, updated, or destroyed when the configuration is applied.
    Ex :
    ```
    resource "aws_instance" "server" {
      count = 4 # create four similar EC2 instances
      ami           = "ami-a1b2c3d4"
      instance_type = "t2.micro"
      tags = {
        Name = "Server ${count.index}"
      }
    }
    ```
  - Count object has one attribute: count.index — The distinct index number (starting with 0) corresponding to this instance.
  - The count meta-argument accepts numeric expressions
  - When count is set, Terraform distinguishes between the block itself and the multiple resource or module instances associated with it. Instances are identified by an index number, starting with 0
  - Within nested provisioner or connection blocks, the special self object refers to the current resource instance, not the resource block as a whole
    
- for_each
  - If a resource or module block includes a for_each argument whose value is a map or a set of strings, Terraform creates one instance for each member of that map or set. Each instance has a distinct infrastructure object associated with it, and each is separately created, updated, or destroyed when the configuration is applied.
  - for_each was added in Terraform 0.12.6. Module support for for_each was added in Terraform 0.13; previous versions can only use it with resources.
  - A given resource or module block cannot use both count and for_each.
  - for_each is a meta-argument defined by the Terraform language. It can be used with modules and with every resource type.
  - In blocks where for_each is set, an additional each object is available in expressions, so you can modify the configuration of each instance. This object has two attributes:
    - each.key   : The map key (or set member) corresponding to this instance.
    - each.value : The map value corresponding to this instance. (If a set was provided, this is the same as each.key.)
  - Limitations on values used in for_each
    - The keys of the map (or all the values in the case of a set of strings) must be known values, or you will get an error message that for_each has dependencies that cannot be determined before apply, and a -target may be needed
    -  for_each keys cannot be the result (or rely on the result of) of impure functions, including uuid, bcrypt, or timestamp, as their evaluation is deferred during the main evaluation step.
    - Sensitive values, such as sensitive input variables, sensitive outputs, or sensitive resource attributes, cannot be used as arguments to for_each. The value used in for_each is used to identify the resource instance and will always be disclosed in UI output, which is why sensitive values are not allowed. Attempts to use sensitive values as for_each arguments will result in an error.
    - If you transform a value containing sensitive data into an argument to be used in for_each, be aware that most functions in Terraform will return a sensitive result if given an argument with any sensitive content. In many cases, you can achieve similar results to a function used for this purpose by using a for expression
      ex: if you would like to call keys(local.map), where local.map is an object with sensitive values (but non-sensitive keys), you can create a value to pass to for_each with toset([for k,v in local.map : k]).
    - A resource using for_each appears as a map of objects when used in expressions elsewhere, you can directly use one resource as the for_each of another in situations where there is a one-to-one relationship between two sets of objects.
    - When for_each is set, Terraform distinguishes between the block itself and the multiple resource or module instances associated with it. Instances are identified by a map key (or set member) from the value provided to for_each.
      - The Terraform language doesn't have a literal syntax for set values, but you can use the toset function to explicitly convert a list of strings to a set
      - each.key and each.value are the same for a set
      - Conversion from list to set discards the ordering of the items in the list and removes any duplicate elements. toset(["b", "a", "b"]) will produce a set containing only "a" and "b" in no particular order; the second "b" is discarded
    
- provider
  - The provider meta-argument specifies which provider configuration to use for a resource, overriding Terraform's default behavior of selecting one based on the resource type name. Its value should be an unquoted <PROVIDER>.<ALIAS> reference
  - you can optionally create multiple configurations for a single provider (usually to manage resources in different regions of multi-region services). Each provider can have one default configuration, and any number of alternate configurations that include an extra name segment (or "alias").
  - A resource always has an implicit dependency on its associated provider, to ensure that the provider is fully configured before any resource actions are taken.
    
- lifecycle
  - The Resource Behavior page describes the general lifecycle for resources. Some details of that behavior can be customized using the special nested lifecycle block within a resource block body.
  - lifecycle is a nested block that can appear within a resource block. The lifecycle block and its contents are meta-arguments, available for all resource blocks regardless of type.
  - The arguments available within a lifecycle block are create_before_destroy, prevent_destroy, ignore_changes, and replace_triggered_by.
    - create_before_destroy (bool)
      - By default, when Terraform must change a resource argument that cannot be updated in-place due to remote API limitations, Terraform will instead destroy the existing object and then create a new replacement object with the new configured arguments.
    - The create_before_destroy meta-argument changes this behavior so that the new replacement object is created first, and the prior object is destroyed after the replacement is created.
      - Terraform propagates and applies the create_before_destroy meta-attribute behaviour to all resource dependencies, You cannot override create_before_destroy to false on dependent resource because that would imply dependency cycles in the graph.
    - Destroy provisioners of this resource do not run if create_before_destroy is set to true.
      - prevent_destroy (bol)
        - This meta-argument, when set to true, will cause Terraform to reject with an error any plan that would destroy the infrastructure object associated with the resource, as long as the argument remains present in the configuration.
      - ignore_changes (list of attribute names)
        - By default, Terraform detects any difference in the current settings of a real infrastructure object and plans to update the remote object to match configuration. The ignore_changes feature is intended to be used when a resource is created with references to data that may change in the future, but should not affect said resource after its creation
      - The ignore_changes meta-argument specifies resource attributes that Terraform should ignore when planning updates to the associated remote object.
        - The arguments corresponding to the given attribute names are considered when planning a create operation, but are ignored when planning an update. The arguments are the relative address of the attributes in the resource. Map and list elements can be referenced using index notation, like tags["Name"] and list[0] respectively
        - Instead of a list, the special keyword all may be used to instruct Terraform to ignore all attributes, which means that Terraform can create and destroy the remote object but will never propose updates to it.
      - replace_triggered_by (list of resource or attribute references)
        - Replaces the resource when any of the referenced items change. Supply a list of expressions referencing managed resources, instances, or instance attributes. When used in a resource that uses count or for_each, you can use count.index or each.key in the expression to reference specific instances of other resources that are configured with the same count or collection.
      - References trigger replacement in the following conditions:
        - If the reference is to a resource with multiple instances, a plan to update or replace any instance will trigger replacement.
        - If the reference is to a single resource instance, a plan to update or replace that instance will trigger replacement.
        - If the reference is to a single attribute of a resource instance, any change to the attribute value will trigger replacement.
    - You can add precondition and postcondition blocks with a lifecycle block to specify assumptions and guarantees about how resources and data sources operate
    - The lifecycle settings all affect how Terraform constructs and traverses the dependency graph. As a result, only literal values can be used because the processing happens too early for arbitrary expression evaluation.

## Provisioners
- You can use provisioners to model specific actions on the local machine or on a remote machine in order to prepare servers or other infrastructure objects for service.
- Terraform includes the concept of provisioners as a measure of pragmatism, knowing that there are always certain behaviors that cannot be directly represented in Terraform's declarative model
- Provisioners also add a considerable amount of complexity and uncertainty to Terraform usage
  1. Terraform cannot model the actions of provisioners as part of a plan because they can in principle take any action
  2. successful use of provisioners requires coordinating many more details than Terraform usage usually requires: direct network access to your servers, issuing Terraform credentials to log in, making sure that all of the necessary external software is installed, etc.
- Provisioning files using cloud-config
  - You can add the cloudinit_config data source to your Terraform configuration and specify the files you want to provision as text/cloud-config content. The cloudinit_config data source renders multi-part MIME configurations for use with cloud-init. Pass the files in the content field as YAML-encoded configurations using the write_files block.
  - It is technically possible to use the local-exec provisioner to run the CLI for your target system in order to create, update, or otherwise interact with remote objects in that system.
  - If the functionality you need is not available in a provider today, we suggest to consider local-exec usage a temporary workaround and to also open an issue in the relevant provider's repository to discuss adding first-class provider support. Provider development teams often prioritize features based on interest, so opening an issue is a way to record your interest in the feature
  - Provisioners are used to execute scripts on a local or remote machine as part of resource creation or destruction. Provisioners can be used to bootstrap a resource, cleanup before destroy, run configuration management, etc.
  - If you are certain that provisioners are the best way to solve your problem after considering the advice in the sections above, you can add a provisioner block inside the resource block of a compute instance.
  ex:
  ```
  resource "aws_instance" "web" {
    provisioner "local-exec" {
      command = "echo The server's IP address is ${self.private_ip}"
    }
  }
  ```
  - The local-exec provisioner requires no other configuration, but most other provisioners must connect to the remote system using SSH or WinRM. You must include a connection block so that Terraform knows how to communicate with the server.
  - we do not recommend using any provisioners except the built-in file, local-exec, and remote-exec provisioners.
  - All provisioners support the when and on_failure meta-arguments
  - Expressions in provisioner blocks cannot refer to their parent resource by name. Instead, they can use the special self object.
  - The self object represents the provisioner's parent resource, and has all of that resource's attributes. For example, use self.public_ip to reference an aws_instance's public_ip attribute.
  - Resource references are restricted here because references create dependencies. Referring to a resource by name within its own block would create a dependency cycle.
  - Creation-Time Provisioners
    - By default, provisioners run when the resource they are defined within is created. Creation-time provisioners are only run during creation, not during updating or any other lifecycle. They are meant as a means to perform bootstrapping of a system.
    - If a creation-time provisioner fails, the resource is marked as tainted. A tainted resource will be planned for destruction and recreation upon the next terraform apply.
    - You can change this behavior by setting the on_failure attribute
  - Destroy-Time Provisioners
    - If when = destroy is specified, the provisioner will run when the resource it is defined within is destroyed.
      ```
        ex:
            resource "aws_instance" "web" {
                provisioner "local-exec" {
                    when    = destroy
                    command = "echo 'Destroy-time provisioner'"
                }
            }
      ```
    - Destroy provisioners are run before the resource is destroyed. If they fail, Terraform will error and rerun the provisioners again on the next terraform apply
    - A destroy-time provisioner within a resource that is tainted will not run. This includes resources that are marked tainted from a failed creation-time provisioner or tainted manually using terraform taint.
  - Multiple Provisioners
    - Multiple provisioners can be specified within a resource. Multiple provisioners are executed in the order they're defined in the configuration file.
    - You may also mix and match creation and destruction provisioners. Only the provisioners that are valid for a given operation will be run. Those valid provisioners will be run in the order they're defined in the configuration file.
  - Failure Behavior
    - By default, provisioners that fail will also cause the Terraform apply itself to fail. The on_failure setting can be used to change this. The allowed values are:
      - continue : Ignore the error and continue with creation or destruction.
      - fail : Raise an error and stop applying (the default behavior). If this is a creation provisioner, taint the resource
    ```
        resource "aws_instance" "web" {
            provisioner "local-exec" {
                command    = "echo The server's IP address is ${self.private_ip}"
                on_failure = continue
            }
        }
    ```
## Provisioner Connections
- Most provisioners require access to the remote resource via SSH or WinRM and expect a nested connection block with details about how to connect
- Connection Block
  - You can create one or more connection blocks that describe how to access the remote resource. One use case for providing multiple connections is to have an initial provisioner connect as the root user to set up user accounts and then have subsequent provisioners connect as a user with more limited permissions.
  - Connection blocks don't take a block label and can be nested within either a resource or a provisioner.
    - A connection block nested directly within a resource affects all of that resource's provisioners.
    - A connection block nested in a provisioner block only affects that provisioner and overrides any resource-level connection settings
  - Provisioners which execute commands on a remote system via a protocol such as SSH typically achieve that by uploading a script file to the remote system and then asking the default shell to execute it. Provisioners use this strategy because it then allows you to use all of the typical scripting techniques supported by that shell, including preserving environment variable values and other context between script statements.

## Provisioners Without a Resource
- If you need to run provisioners that aren't directly associated with a specific resource, you can associate them with a terraform_data.
- Instances of terraform_data are treated like normal resources, but they don't do anything. Like with any other resource type, you can configure provisioners and connection details on a terraform_data resource. You can also use its input argument, triggers_replace argument, and any meta-arguments to control exactly where in the dependency graph its provisioners will run.

## File Provisioner
- The file provisioner copies files or directories from the machine running Terraform to the newly created resource. The file provisioner supports both ssh and winrm type connections.
- When the file provisioner communicates with a Windows system over SSH, you must configure OpenSSH to run the commands with cmd.exe and not PowerShell. PowerShell causes file parsing errors because it is incompatible with both Unix shells and the Windows command interpreter.
- The following arguments are supported:
  - source : The source file or directory. Specify it either relative to the current working directory or as an absolute path. This argument cannot be combined with content.
  - content : The direct content to copy on the destination. If destination is a file, the content will be written on that file. In case of a directory, a file named tf-file-content is created inside that directory. We recommend using a file as the destination when using content. This argument cannot be combined with source.
  - destination : (Required) The destination path to write to on the remote system. See Destination Paths below for more information.
- The path you provide in the destination argument will be evaluated by the remote system, rather than by Terraform itself. Therefore the valid values for that argument can vary depending on the operating system and remote access software running on the target.
- The remote scp process will run with the access level of the user specified in the connection block, and so permissions may prevent writing directly to locations outside of the home directory.
- The file provisioner can upload a complete directory to the remote machine. When uploading a directory, there are some additional considerations. When using the ssh connection type the destination directory must already exist. If you need to create it, use a remote-exec provisioner just prior to the file provisioner in order to create the directory

## local-exec Provisioner
- The local-exec provisioner invokes a local executable after a resource is created. This invokes a process on the machine running Terraform, not on the resource.
- The following arguments are supported:
  1. command (Required)
     - This is the command to execute. It can be provided as a relative path to the current working directory or as an absolute path. The command is is evaluated in a shell and can use environment variables for variable substitution. We do not recommend using Terraform variables for variable substitution because doing so can lead to shell injection vulnerabilities. Instead, you should pass Terraform variables to a command through the environment parameter and use environment variable substitution instead
  2. working_dir
  3. interpreter
  4. environment
     - block of key value pairs representing the environment of the executed command. inherits the current process environment.
  5. when - If provided, specifies when Terraform will execute the command.
  6. quiet 
     - if set to true, Terraform will not print the command to be executed to stdout, and will instead print "Suppressed by quiet=true". Note that the output of the command will still be printed in any case.

## remote-exec Provisioner
- The remote-exec provisioner invokes a script on a remote resource after it is created. This can be used to run a configuration management tool, bootstrap into a cluster, etc. To invoke a local process, see the local-exec provisioner instead. The remote-exec provisioner requires a connection and supports both ssh and winrm
- The following arguments are supported:
  1. inline 
     - This is a list of command strings. The provisioner uses a default shell unless you specify a shell as the first command (eg., #!/bin/bash). You cannot provide this with script or scripts.
  2. script 
     - This is a path (relative or absolute) to a local script that will be copied to the remote resource and then executed. This cannot be provided with inline or scripts.
  3. scripts 
     - This is a list of paths (relative or absolute) to local scripts that will be copied to the remote resource and then executed. They are executed in the order they are provided. This cannot be provided with inline or script.
- Since inline is implemented by concatenating commands into a script, on_failure applies only to the final command in the list. In particular, with on_failure = fail (the default behaviour) earlier commands will be allowed to fail, and later commands will also execute. If this behaviour is not desired, consider using "set -o errexit" as the first command.
- You cannot pass any arguments to scripts using the script or scripts arguments to this provisioner. If you want to specify arguments, upload the script with the file provisioner and then use inline to call it. 
Example:
```
        resource "aws_instance" "web" {
            provisioner "file" {
                source      = "script.sh"
                destination = "/tmp/script.sh"
            }

            provisioner "remote-exec" {
                inline = [
                    "chmod +x /tmp/script.sh",
                    "/tmp/script.sh args",
                ]
            }
        }
```
## The terraform_data Managed Resource Type
- The terraform_data implements the standard resource lifecycle, but does not directly take any other actions. You can use the terraform_data resource without requiring or configuring a provider. It is always available through a built-in provider with the source address terraform.io/builtin/terraform.
  - The terraform_data resource is useful for storing values which need to follow a manage resource lifecycle, and for triggering provisioners when there is no other logical managed resource in which to place them.
  - The following arguments are supported:
    1. input - (Optional) 
       - A value which will be stored in the instance state, and reflected in the output attribute after apply.
    2. triggers_replace - (Optional) 
       - A value which is stored in the instance state, and will force replacement when the value changes.
  - the following attributes are exported:
    1. id 
       - A string value unique to the resource instance.
    2. output 
       - The computed value derived from the input argument. During a plan where output is unknown, it will still be of the same type as input
