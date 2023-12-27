Source - https://developer.hashicorp.com/terraform/language

# INTRODUCTION
- Terraform's language is its primary user interface. Configuration files you write in Terraform language tell Terraform what plugins to install, what infrastructure to create, and what data to fetch. Terraform language also lets you define dependencies between resources and create multiple similar resources from a single configuration block.
- The main purpose of the Terraform language is declaring resources, which represent infrastructure objects. All other language features exist only to make the definition of resources more flexible and convenient.
- A Terraform configuration is a complete document in the Terraform language that tells Terraform how to manage a given collection of infrastructure. A configuration can consist of multiple files and directories.
  ```
  <BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
            # Block body
            <IDENTIFIER> = <EXPRESSION> # Argument
        }
  ```
  - Blocks : are containers for other content and usually represent the configuration of some kind of object, like a resource. Blocks have a block type, can have zero or more labels, and have a body that contains any number of arguments and nested blocks. Most of Terraform's features are controlled by top-level blocks in a configuration file.
  - Arguments : assign a value to a name. They appear within blocks.
  - Expressions : represent a value, either literally or by referencing and combining other values. They appear as values for arguments, or within other expressions
- The Terraform language is declarative, describing an intended goal rather than the steps to reach that goal. The ordering of blocks and the files they are organized into are generally not significant; Terraform only considers implicit and explicit relationships between resources when determining an order of operations

# FILES AND DIRECTORIES
- Code in the Terraform language is stored in plain text files with the .tf file extension. There is also a JSON-based variant of the language that is named with the .tf.json file extension. Files containing Terraform code are often called configuration files **
- Configuration files must always use UTF-8 encoding, and by convention usually use Unix-style line endings (LF) rather than Windows-style line endings (CRLF), though both are accepted.
- A module is a collection of .tf and/or .tf.json files kept together in a directory. A Terraform module only consists of the top-level configuration files in a directory; nested directories are treated as completely separate modules, and are not automatically included in the configuration. Terraform evaluates all of the configuration files in a module, effectively treating the entire module as a single document
- Terraform always runs in the context of a single root module. A complete Terraform configuration consists of a root module and the tree of child modules
  1. Terraform normally loads all of the .tf and .tf.json files within a directory and expects each one to define a distinct set of configuration objects. If two files attempt to define the same object, Terraform returns an error. **
  2. In some rare cases, it is convenient to be able to override specific portions of an existing configuration object in a separate file. For example, a human-edited configuration file in the Terraform language native syntax could be partially overridden using a programmatically-generated file in JSON syntax. For these rare situations, Terraform has special handling of any configuration file whose name ends in _override.tf or _override.tf.json. This special handling also applies to a file named literally override.tf or override.tf.json. **
  3. Terraform initially skips these override files when loading configuration, and then afterwards processes each one in turn (in lexicographical order). For each top-level block defined in an override file, Terraform attempts to find an already-defined object corresponding to that block and then merges the override block contents into the existing object **
    
- The merging behavior is slightly different for each block type, and some special constructs within certain blocks are merged in a special way. General Rule,
  1. A top-level block in an override file merges with a block in a normal configuration file that has the same block header. The block header is the block type and any quoted labels that follow it.
  2. Within a top-level block, an attribute argument within an override block replaces any argument of the same name in the original block.
  3. Within a top-level block, any nested blocks within an override block replace all blocks of the same type in the original block. Any block types that do not appear in the override block remain from the original block.
  4. The contents of nested configuration blocks are not merged.
  5. The resulting merged block must still comply with any validation rules that apply to the given block type.
    
- If more than one override file defines the same top-level block, the overriding effect is compounded, with later blocks taking precedence over earlier blocks. Overrides are processed in order first by filename (in lexicographical order) and then by position in each file.
    - Resource and Data Block override
        - Within a resource block, the contents of any lifecycle nested block are merged on an argument-by-argument basis
        - If an overriding resource block contains one or more provisioner blocks then any provisioner blocks in the original block are ignored.
        - If an overriding resource block contains a connection block then it completely overrides any connection block present in the original block.
        - The depends_on meta-argument may not be used in override blocks, and will produce an error
    - Variable block override
        - The arguments within a variable block are merged in the standard way 
        - some special considerations apply due to the interactions between the type and default arguments.If the original block defines a default value and an override block changes the variable's type, Terraform attempts to convert the default value to the overridden type, producing an error if this conversion is not possible.
        - Conversely, if the original block defines a type and an override block changes the default, the overridden default value must be compatible with the original type specification
    - Output Block override
        - The depends_on meta-argument may not be used in override blocks, and will produce an error.
    - locals Block override
        - Each locals block defines a number of named values. Overrides are applied on a value-by-value basis, ignoring which locals block they are defined in.
    - terraform block override 
        - The settings within terraform blocks are considered individually when merging.
        - If the required_providers argument is set, its value is merged on an element-by-element basis, which allows an override block to adjust the constraint for a single provider without affecting the constraints for other providers.
        - In both the required_version and required_providers settings, each override constraint entirely replaces the constraints for the same component in the original block. If both the base block and the override block both set required_version then the constraints in the base block are entirely ignored
        - The presence of a block defining a backend (either cloud or backend) in an override file always takes precedence over a block defining a backend in the original configuration. That is, if a cloud block is set within the original configuration and a backend block is set in the override file, Terraform will use the backend block specified in the override file upon merging. Similarly, if a backend block is set within the original configuration and a cloud block is set in the override file, Terraform will use the cloud block specified in the override file upon merging.
    - Dependency Lock file
        - A Terraform configuration may refer to two different kinds of external dependency that come from outside of its own codebase:
           - Providers
                which are plugins for Terraform that extend it with support for interacting with various external systems.
           - Modules
                which allow splitting out groups of Terraform configuration constructs (written in the Terraform language) into reusable abstractions.
        - Version constraints within the configuration itself determine which versions of dependencies are potentially compatible, but after selecting a specific version of each dependency Terraform remembers the decisions it made in a dependency lock file so that it can (by default) make the same decisions again in future.
        - The dependency lock file is a file that belongs to the configuration as a whole, rather than to each separate module in the configuration. For that reason Terraform creates it and expects to find it in your current working directory when you run Terraform, which is also the directory containing the .tf files for the root module of your configuration
        - The lock file is always named .terraform.lock.hcl, and this name is intended to signify that it is a lock file for various items that Terraform caches in the .terraform subdirectory of your working directory.
        - The dependency lock file uses the same low-level syntax as the main Terraform language, but the dependency lock file is not itself a Terraform language configuration file. It is named with the suffix .hcl instead of .tf in order to signify that difference
        - If a particular provider has no existing recorded selection, Terraform will select the newest available version that matches the given version constraint, and then update the lock file to include that selection
        - If a particular provider already has a selection recorded in the lock file, Terraform will always re-select that version for installation, even if a newer version has become available. You can override that behavior by adding the -upgrade option when you run terraform init, in which case Terraform will disregard the existing selections and once again select the newest available version matching the version constraint.
        - Terraform will also verify that each package it installs matches at least one of the checksums it previously recorded in the lock file, if any, returning an error if none of the checksums match:
        - Provider will have below entry records
            - version
            - constraints
            - hashes
        - The addition of a new checksum into the hashes value represents Terraform gradually transitioning between different hashing schemes. The h1: and zh: prefixes on these values represent different hashing schemes, each of which represents calculating a checksum using a different algorithm. We may occasionally introduce new hashing schemes if we learn of limitations in the existing schemes or if a new scheme offers some considerable additional benefit.
            - zh:: a mnemonic for "zip hash", this is a legacy hash format which is part of the Terraform provider registry protocol and is therefore used for providers that you install directly from an origin registry.
            - h1:: a mnemonic for "hash scheme 1", which is the current preferred hashing scheme.
        - When installing a particular provider for the first time (where there is no existing provider block for it), Terraform will pre-populate the hashes value with any checksums that are covered by the provider developer's cryptographic signature, which usually covers all of the available packages for that provider version across all supported platforms
        - If you wish to avoid ongoing additions of new h1: hashes as you work with your configuration on new target platforms, or if you are installing providers from a mirror that therefore can't provide official signed checksums, you can ask Terraform to pre-populate hashes for a chosen set of platforms using the terraform providers lock command:
            - terraform providers lock -platform=linux_arm64 -platform=linux_amd64 -platform=darwin_amd64 -platform=windows_amd64
        - Specific Terraform commands, such as test, init, and validate, load Terraform test files for your configuration.
        - Terraform test files use the file extensions .tftest.hcl and .tftest.json
        - Terraform loads all test files within your root configuration directory.
        - Terraform also loads the test files within the testing directory. You can override the location of the testing directory by appending the -test-directory flag to commands that load test files. The default testing directory is tests relative to your configuration directory.

# SYNTAX 
- The majority of the Terraform language documentation focuses on the practical uses of the language and the specific constructs it uses
  - Configuration Syntax:
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

      - JSON Configuration Syntax: 
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
      - Style Conventions :
        - documents some commonly accepted formatting guidelines for Terraform code. These conventions can be enforced automatically with terraform fmt.
          - Indent two spaces for each nesting level
          - Use empty lines to separate logical groups of arguments within a block
          - For blocks that contain both arguments and "meta-arguments" (as defined by the Terraform language semantics), list meta-arguments first and separate them from other arguments with one blank line. Place meta-argument blocks last and separate them from other blocks with one blank line.
          - Avoid grouping multiple blocks of the same type with other blocks of a different type, unless the block types are defined by semantics to form a family.
          - 
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

# DATA SOURCES
- Data sources allow Terraform to use information defined outside of Terraform, defined by another separate Terraform configuration, or modified by functions
- Each provider may offer data sources alongside its set of resource types.
- A data source is accessed via a special kind of resource known as a data resource, declared using a data block:
ex:
```
        data "aws_ami" "example" {
            most_recent = true
            owners = ["self"]
            tags = {
                Name   = "app-server"
                Tested = "true"
            }
        }
```
- A data block requests that Terraform read from a given data source ("aws_ami") and export the result under the given local name ("example"). The name is used to refer to this resource from elsewhere in the same Terraform module, but has no significance outside of the scope of a module
- The data source and name together serve as an identifier for a given resource and so must be unique within a module.
- When distinguishing from data resources, the primary kind of resource (as declared by a resource block) is known as a managed resource. Both kinds of resources take arguments and export attributes for use in configuration, but while managed resources cause Terraform to create, update, and delete infrastructure objects, data resources cause Terraform only to read objects
  - Most of the items within the body of a data block are defined by and specific to the selected data source, and these arguments can make full use of expressions and other dynamic Terraform language features
  - Terraform reads data resources during the planning phase when possible, but announces in the plan when it must defer reading resources until the apply phase to preserve the order of operations
  - Terraform defers reading data resources in the following situations:
    1. At least one of the given arguments is a managed resource attribute or other value that Terraform cannot predict until the apply step.
    2. The data resource depends directly on a managed resource that itself has planned changes in the current plan.
    3. The data resource has custom conditions and it depends directly or indirectly on a managed resource that itself has planned changes in the current plan

- While many data sources correspond to an infrastructure object type that is accessed via a remote network API, some specialized data sources operate only within Terraform itself, calculating some results and exposing them for use elsewhere. For example, local-only data sources exist for rendering templates, reading local files.
- The behavior of local-only data sources is the same as all other data sources, but their result data exists only temporarily during a Terraform operation, and is re-calculated each time a new plan is created.
- Due to the data resource behavior of deferring the read until the apply phase when depending on values that are not yet known, using depends_on with data resources will force the read to always be deferred to the apply phase, and therefore a configuration that uses depends_on with a data resource can never converge. Due to this behavior, we do not recommend using depends_on with data resources
- You can use precondition and postcondition blocks to specify assumptions and guarantees about how the data source operates
- Data resources support count and for_each meta-arguments as defined for managed resources, with the same syntax and behavior.
- As with managed resources, when count or for_each is present it is important to distinguish the resource itself from the multiple resource instances it creates. Each instance will separately read from its data source with its own variant of the constraint arguments, producing an indexed result
- Data resources support the provider meta-argument as defined for managed resources, with the same syntax and behavior.
- Data resources do not have any customization settings available for their lifecycle. However, the lifecycle block is reserved for future versions.
- Each data instance will export one or more attributes, which can be used in other resources as reference expressions of the form data.<TYPE>.<NAME>.<ATTRIBUTE>.

## Data Source Lifecycle
- If the arguments of a data instance contain no references to computed values, such as attributes of resources that have not yet been created, then the data instance will be read and its state updated during Terraform's "refresh" phase, which by default runs prior to creating a plan. This ensures that the retrieved data is available for use during planning and the diff will show the real values obtained.

# PROVIDERS
- Terraform relies on plugins called providers to interact with cloud providers, SaaS providers, and other APIs. Terraform configurations must declare which providers they require so that Terraform can install and use them. Additionally, some providers require configuration (like endpoint URLs or cloud regions) before they can be used.
- Each provider adds a set of resource types and/or data sources that Terraform can manage. Every resource type is implemented by a provider; without providers, Terraform can't manage any kind of infrastructure.
- Providers are distributed separately from Terraform itself, and each provider has its own release cadence and version numbers. The Terraform Registry is the main directory of publicly available Terraform providers, and hosts providers for most major infrastructure platforms.
## Provider Installation
- Terraform Cloud and Terraform Enterprise install providers as part of every run.
- Terraform CLI finds and installs providers when initializing a working directory. It can automatically download providers from a Terraform registry, or load them from a local mirror or cache. If you are using a persistent working directory, you must reinitialize whenever you change a configuration's providers
- To save time and bandwidth, Terraform CLI supports an optional plugin cache. You can enable the cache using the plugin_cache_dir setting in the CLI configuration file.
- To ensure Terraform always installs the same provider versions for a given configuration, you can use Terraform CLI to create a dependency lock file and commit it to version control along with your configuration. If a lock file is present, Terraform Cloud, CLI, and Enterprise will all obey it when installing providers.
## Provider Configuration
- A provider configuration is created using a provider block:
ex:
```
        provider "google" {
            project = "acme-app"
            region  = "us-central1"
        }
```
- The name given in the block header ("google" in this example) is the local name of the provider to configure. This provider should already be included in a required_providers block.
- There are also two "meta-arguments" that are defined by Terraform itself and available for all provider blocks:
  1. alias
     - for using the same provider with different configurations for different resources version, which we no longer recommend (use provider requirements instead) 
     ex: Multiple regions 
     - To declare a configuration alias within a module in order to receive an alternate provider configuration from the parent module, add the configuration_aliases argument to that provider's required_providers entry
  2. version
     - which we no longer recommend (use provider requirements instead)
- A provider block without an alias argument is the default configuration for that provider. Resources that don't set the provider meta-argument will use the default provider configuration that matches the first word of the resource type name
- When Terraform needs the name of a provider configuration, it expects a reference of the form <PROVIDER NAME>.<ALIAS> , These references are special expressions.  Like references to other named entities (for example, var.image_id), they aren't strings and don't need to be quoted. But they are only valid in specific meta-arguments of resource, data, and module blocks, and can't be used in arbitrary expressions.
- To use an alternate provider configuration for a resource or data source, set its provider meta-argument to a <PROVIDER NAME>.<ALIAS> reference
- To select alternate provider configurations for a child module, use its providers meta-argument to specify which provider configurations should be mapped to which local provider names inside the module
## Provider Requirements
- Each Terraform module must declare which providers it requires, so that Terraform can install and use them. Provider requirements are declared in a required_providers block.
- A provider requirement consists of a local name, a source location, and a version constraint
```
  terraform {
    required_providers {
      mycloud = {
        source  = "mycorp/mycloud"
        version = "~> 1.0"
      }
    }
  }
```
- The required_providers block must be nested inside the top-level terraform block (which can also contain other settings).
- Each argument in the required_providers block enables one provider. The key determines the provider's local name.
- Each provider has two identifiers:
  1. A unique source address, which is only used when requiring a provider.
  2. A local name, which is used everywhere else in a Terraform module.
- Local names are module-specific, and are assigned when requiring a provider. Local names must be unique per-module
- A provider's source address is its global identifier. It also specifies the primary location where Terraform can download it. Source addresses consist of three parts delimited by slashes (/), as follows:
        [<HOSTNAME>/]<NAMESPACE>/<TYPE>   # Hostname (optional)
  ## Dependency Lock File
  - At present, the dependency lock file tracks only provider dependencies. Terraform does not remember version selections for remote modules, and so Terraform will always select the newest available module version that meets the specified version constraints. You can use an exact version constraint to ensure that Terraform will always select the same module version
  - The dependency lock file is a file that belongs to the configuration as a whole, rather than to each separate module in the configuration. For that reason Terraform creates it and expects to find it in your current working directory when you run Terraform, which is also the directory containing the .tf files for the root module of your configuration.
  - The lock file is always named .terraform.lock.hcl, and this name is intended to signify that it is a lock file for various items that Terraform caches in the .terraform subdirectory of your working directory.
  - Terraform automatically creates or updates the dependency lock file each time you run the terraform init command.
  - The dependency lock file uses the same low-level syntax as the main Terraform language, but the dependency lock file is not itself a Terraform language configuration file. It is named with the suffix .hcl instead of .tf in order to signify that difference.
  - If a particular provider already has a selection recorded in the lock file, Terraform will always re-select that version for installation, even if a newer version has become available. You can override that behavior by adding the -upgrade option when you run terraform init, in which case Terraform will disregard the existing selections and once again select the newest available version matching the version constraint.

# VARIABLES & OUTPUTS
## Input Variables : 
  - serve as parameters for a Terraform module, so users can customize behavior without editing the source.
  - Input variables let you customize aspects of Terraform modules without altering the module's own source code. This functionality allows you to share modules across different Terraform configurations, making your module composable and reusable
  - Input variables are like function arguments
  - For brevity, input variables are often referred to as just "variables" or "Terraform variables" when it is clear from context what sort of variable is being discussed. Other kinds of variables in Terraform include environment variables (set by the shell where Terraform runs) and expression variables (used to indirectly represent a value in an expression).
  - The label after the variable keyword is a name for the variable, which must be unique among all variables in the same module
  ```
    variable "image_id" {
      type = string
    }
  ```
  - The name of a variable can be any valid identifier except these: source, version, providers, count, for_each, lifecycle, depends_on, locals
  - Terraform CLI defines the following optional arguments for variable declarations:
    - default       : A default value which then makes the variable optional.
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
    2. Individually, with the -var command line option.
    3. In variable definitions (.tfvars or .tfvars.json) files, either specified on the command line or automatically loaded. (-var-file)
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
  - Within Terraform test files, you can specify variable values within variables blocks, either nested within run blocks or defined directly within the file. Variables defined in this way take precedence over all other mechanisms during test execution, with variables defined within run blocks taking precedence over those defined within the file

## Output Values :  
  - Output values are like function return values or return values for a Terraform module.
  - 
## Local Values : 
  - Local values are like a function's temporary local variables, convenience feature for assigning a short name to an expression.
  - 









