- Modules are containers for multiple resources that are used together. A module consists of a collection of .tf and/or .tf.json files kept together in a directory.
- Modules are the main way to package and reuse resource configurations with Terraform.
- Every Terraform configuration has at least one module, known as its root module, which consists of the resources defined in the .tf files in the main working directory.
- A Terraform module (usually the root module of a configuration) can call other modules to include their resources into the configuration. A module that has been called by another module is often referred to as a child module
- In addition to modules from the local filesystem, Terraform can load modules from a public or private registry. This makes it possible to publish modules for others to use, and to use modules that others have published.
- Using Modules.
    - Module Blocks documents the syntax for calling a child module from a parent module, including meta-arguments like for_each.
    - Module Sources documents what kinds of paths, addresses, and URIs can be used in the source argument of a module block.
    - The Meta-Arguments section documents special arguments that can be used with every module, including providers, depends_on, count, and for_each.

## Module Blocks
- To call a module means to include the contents of that module into the configuration with specific values for its input variables. Modules are called from within other modules using `module` blocks:
```
module "servers" {
  source = "./app-cluster"

  servers = 5
}

```
- A module that includes a `module` block like this is the calling module of the child module
- The label immediately after the `module` keyword is a local name, which the calling module can use to refer to this instance of the module
- Module calls use the following kinds of arguments:
    1. The `source` argument is mandatory for all modules.
        - All modules require a `source` argument, which is a meta-argument defined by Terraform. Its value is either the path to a local directory containing the module's configuration files, or a remote module source that Terraform should download and use. This value must be a literal string with no template sequences; arbitrary expressions are not allowed
        - The same source address can be specified in multiple `module` blocks to create multiple copies of the resources defined within, possibly with different variable values
        - After adding, removing, or modifying `module` blocks, you must re-run `terraform init` to allow Terraform the opportunity to adjust the installed modules. By default this command will not upgrade an already-installed module; use the `-upgrade` option to instead upgrade to the newest available version
    2. The `version` argument is recommended for modules from a registry.
        - The `version` argument accepts a version constraint string. Terraform will use the newest installed version of the module that meets the constraint; if no acceptable versions are installed, it will download the newest version that meets the constraint.
        - Modules sourced from local file paths do not support `version`; since they're loaded from the same source repository, they always share the same version as their caller.
    3. Most other arguments correspond to input variables defined by the module. (The servers argument in the example above is one of these.)
    4. Terraform defines a few other meta-arguments that can be used with all modules, including `for_each` and `depends_on`.
        - Along with `source` and `version`, Terraform defines a few more optional meta-arguments
        - Terraform does not use the `lifecycle` argument. However, the `lifecycle` block is reserved for future versions
- The resources defined in a module are encapsulated, so the calling module cannot access their attributes directly. However, the child module can declare output values to selectively export certain values to be accessed by the calling module.
- You may have an object that needs to be replaced with a new object for a reason that isn't automatically visible to Terraform, such as if a particular virtual machine is running on degraded underlying hardware. In this case, you can use the -replace=... planning option to force Terraform to propose replacing that object.
    ex: `$terraform plan -replace=module.example.aws_instance.example`
- Replacing is a very disruptive action, Terraform only allows selecting individual resource instances. There is no syntax to force replacing all resource instances belonging to a particular module.

## Module Sources
- The `source` argument in a module block tells Terraform where to find the source code for the desired child module.
- Module source addresses use a URL-like syntax, but with extensions to support unambiguous selection of sources and additional features.
- The module installer supports installation from a number of different source types.
    1. Local paths
        - Local path references allow for factoring out portions of a configuration within a single source repository.
        ```
        module "consul" {
            source = "./consul"
        }
        ```
        - A local path must begin with either ./ or ../ to indicate that a local path is intended, to distinguish from a module registry address.
        - Terraform does not consider an absolute filesystem path to be a local path
    2. Terraform Registry
        - A module registry is the native way of distributing Terraform modules for use across multiple configurations, using a Terraform-specific protocol that has full support for module versioning.
        - Modules on the public Terraform Registry can be referenced using a registry source address of the form `<NAMESPACE>/<NAME>/<PROVIDER>`, with each module's information page on the registry site including the exact address to use.
        ```
        module "consul" {
            source = "hashicorp/consul/aws"
            version = "0.1.0"
        }
        ```
        - For modules hosted in other registries, prefix the source address with an additional `<HOSTNAME>/` portion, giving the hostname of the private registry
        ```
        module "consul" {
            source = "app.terraform.io/example-corp/k8s-cluster/azurerm"
            version = "1.1.0"
        }
        ```
        - Both Terraform Cloud and self-hosted Terraform Enterprise support a generic hostname localterraform.com
    3. GitHub
        - Terraform will recognize unprefixed github.com URLs and interpret them automatically as Git repository sources.
        ```
        module "consul" {
            source = "github.com/hashicorp/example"
        }
        ```
        - The above address scheme will clone over HTTPS. To clone over SSH, use the following form:
        ```
        module "consul" {
            source = "git@github.com:hashicorp/example.git"
        }
        ```
    4. Bitbucket
        - Terraform will recognize unprefixed bitbucket.org URLs and interpret them automatically as BitBucket repositories
        ```
        module "consul" {
            source = "bitbucket.org/hashicorp/terraform-consul-aws"
        }
        ```
    5. Generic Git, Mercurial repositories
        - Arbitrary Git repositories can be used by prefixing the address with the special git:: prefix. After this prefix, any valid Git URL can be specified to select one of the protocols supported by Git.
        ```
        module "vpc" {
            source = "git::https://example.com/vpc.git"
        }

        module "storage" {
            source = "git::ssh://username@example.com/storage.git"
        }
        ```
        - Terraform installs modules from Git repositories by running git clone, and so it will respect any local Git configuration set on your system, including credentials. To access a non-public Git repository, configure Git with suitable credentials for that repository
        - By default, Terraform will clone and use the default branch (referenced by HEAD) in the selected repository. You can override this using the ref argument. The value of the ref argument can be any reference that would be accepted by the git checkout command, such as branch, SHA-1 hash (short or full), or tag names
        ```
        # select a specific tag
        module "vpc" {
            source = "git::https://example.com/vpc.git?ref=v1.2.0"
        }

        # directly select a commit using its SHA-1 hash
        module "storage" {
            source = "git::https://example.com/storage.git?ref=51d462976d84fdea54b47d80dcabbf680badcdb8"
        }
        ```
        - The `depth` URL argument corresponds to the `--depth` argument to git clone, telling Git to create a shallow clone with the history truncated to only the specified number of commits.
        - scp-like address.
        ```
        module "storage" {
            source = "git::username@example.com:storage.git"
        }
        ```
        - You can use arbitrary Mercurial repositories by prefixing the address with the special `hg::` prefix. After this prefix, any valid Mercurial URL can be specified to select one of the protocols supported by Mercurial and You can select a non-default branch or tag using the optional `ref` argument

    6. HTTP URLs
    7. S3 buckets
        - You can use archives stored in S3 as module sources using the special `s3::` prefix, followed by an S3 bucket object URL.
    8. GCS buckets
        - You can use archives stored in Google Cloud Storage as module sources using the special `gcs::` prefix, followed by a GCS bucket object URL.
    9. Modules in Package Sub-directories
        - When the source of a module is a version control repository or archive file (generically, a "package"), the module itself may be in a sub-directory relative to the root of the package.
        - A special double-slash syntax is interpreted by Terraform to indicate that the remaining path after that point is a sub-directory within the package

## Meta-Arguments
- Providers 
    ```
    # The default "aws" configuration is used for AWS resources in the root
    # module where no explicit provider instance is selected.
    provider "aws" {
    region = "us-west-1"
    }

    # An alternate configuration is also defined for a different
    # region, using the alias "usw2".
    provider "aws" {
    alias  = "usw2"
    region = "us-west-2"
    }

    # An example child module is instantiated with the alternate configuration,
    # so any AWS resources it defines will use the us-west-2 region.
    module "example" {
    source    = "./example"
    providers = {
        aws = aws.usw2
    }
    }
    ```
    - There are two main reasons to use the `providers` argument:
        1. Using different default provider configurations for a child module.
        2. Configuring a module that requires multiple configurations of the same provider.
    - Terraform configurations that use multiple configurations of the same provider, you might want some child modules to use the default provider configuration and other ones to use an alternate. (This usually happens when using one configuration to manage resources in multiple different regions of the same cloud provider.)
    - By using the `providers` argument (like in the code example above), you can accommodate this without needing to edit the child module. Although the code within the child module always refers to the default provider configuration, the actual configuration of that default can be different for each instance.

- depends_on
    - Use the `depends_on` meta-argument to handle hidden resource or module dependencies that Terraform cannot automatically infer
- count
    - If a resource or module block includes a `count` argument whose value is a whole number, Terraform will create that many instances
    - `count` is a meta-argument defined by the Terraform language. It can be used with modules and with every resource type
- for_each
    - If a resource or module block includes a `for_each` argument whose value is a map or a set of strings, Terraform creates one instance for each member of that map or set

## Module Development
- A module is a container for multiple resources that are used together. You can use modules to create lightweight abstractions, so that you can describe your infrastructure in terms of its architecture, rather than directly in terms of physical objects
- Commonly, to create  Modules use input variables, output variables and resource
- You can also create no-code ready modules to enable the no-code provisioning workflow in Terraform Cloud. No-code provisioning lets users deploy a module's resources in Terraform Cloud without writing any Terraform configuration

### Standard Module Structure
- The standard module structure expects the layout documented below. The list may appear long, but everything is optional except for the root module. Most modules don't need to do any extra work to follow the standard structure
    1. Root module
    2. README
    3. LICENSE
    4. main.tf, variables.tf, outputs.tf  - These are the recommended filenames for a minimal module, even if they're empty.
    5. Variables and outputs should have descriptions - All variables and outputs should have one or two sentence descriptions that explain their purpose
    6. Nested modules - Nested modules should exist under the modules/ subdirectory. Any nested module with a README.md is considered usable by an external user
    7. Examples - Examples of using the module should exist under the examples/ subdirectory at the root of the repository, Each example may have a README to explain the goal and usage of the example

### Providers Within Modules
- Each resource in the configuration must be associated with one provider configuration. Provider configurations, unlike most other concepts in Terraform, are global to an entire Terraform configuration and can be shared across module boundaries. Provider configurations can be defined only in a root Terraform module
- Providers can be passed down to descendent modules in two ways: either implicitly through inheritance, or explicitly via the `providers` argument within a `module` block
- A module intended to be called by one or more other modules must not contain any `provider` blocks. A module containing its own `provider` configurations is not compatible with the `for_each`, `count`, and `depends_on` arguments that were introduced in Terraform v0.13
- To declare multiple configuration names for a provider within a module, add the `configuration_aliases` argument
- Once the `providers` argument is used in a `module` block, it overrides all of the default inheritance behavior, so it is necessary to enumerate mappings for all of the required providers
- If the new version of the module declares `configuration_aliases`, or if the calling module needs the child module to use different provider configurations than its own default provider configurations, the calling module must then include an explicit `providers` argument to describe which provider configurations the child module will use
- Since the association between `resources` and `provider` configurations is static, module calls using `for_each` or `count` cannot pass different provider configurations to different instances. If you need different instances of your module to use different provider configurations then you must use a separate `module` block for each distinct set of provider configurations

### Module Composition
- When we introduce `module` blocks, our configuration becomes hierarchical rather than flat
- If you've built a module that you intend to be reused, we recommend publishing the module on the Terraform Registry. This will version your module, generate documentation, and more
- Explicit refactoring declarations with `moved` blocks is available in Terraform v1.1 and later. For earlier Terraform versions or for refactoring actions too complex to express as `moved` blocks, you can use the `terraform state mv` CLI command as a separate step.
- Terraform compares previous state with new configuration, correlating by each module or resource's unique address. Therefore by default Terraform understands moving or renaming an object as an intent to destroy the object at the old address and to create a new object at the new address.
- When you add `moved` blocks in your configuration to record where you've historically moved or renamed an object, Terraform treats an existing object at the old address as if it now belongs to the new address.
- Each resource type has a separate schema and so objects of different types are not compatible. Therefore, although you can use `moved` to change the name of a resource, you cannot use `moved` to change to a different resource type or to change a managed resource (a `resource` block) into a data resource (a `data` block).
- A `moved` block expects no labels and contains only `from` and `to` arguments
```
moved {
  from = aws_instance.a
  to   = aws_instance.b
}
```
- The example above records that the resource currently known as `aws_instance.b` was known as `aws_instance.a` in a previous version of this module.
- Before creating a new plan for `aws_instance.b`, Terraform first checks whether there is an existing object for `aws_instance.a` recorded in the state. If there is an existing object, Terraform renames that object to `aws_instance.b` and then proceeds with creating a plan. The resulting plan is as if the object had originally been created at `aws_instance.b`, avoiding any need to destroy it during apply.
- The `from` and `to` addresses both use a special addressing syntax that allows selecting modules, resources, and resources inside child modules
- Removing a `moved` block is a generally breaking change because any configurations that refer to the old address will plan to delete that existing object instead of move it. We strongly recommend that you retain all historical `moved` blocks from earlier versions of your modules to preserve the upgrade path for users of any previous version. If you do decide to remove `moved` blocks, proceed with caution. It can be safe to remove `moved` blocks when you are maintaining private modules within an organization and you are certain that all users have successfully run `terraform apply` with your new module version
