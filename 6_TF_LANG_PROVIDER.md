# PROVIDERS
- Terraform relies on plugins called providers to interact with cloud providers, SaaS providers, and other APIs. Terraform configurations must declare which providers they require so that Terraform can install and use them. Additionally, some providers require configuration (like endpoint URLs or cloud regions) before they can be used.
- Each provider adds a set of resource types and/or data sources that Terraform can manage. Every resource type is implemented by a provider; without providers, Terraform can't manage any kind of infrastructure.
- Providers are distributed separately from Terraform itself, and each provider has its own release cadence and version numbers. The Terraform Registry is the main directory of publicly available Terraform providers, and hosts providers for most major infrastructure platforms.
## Provider Installation
- Terraform Cloud and Terraform Enterprise install providers as part of every run.
- Terraform CLI finds and installs providers when initializing a working directory. It can automatically download providers from a Terraform registry, or load them from a local mirror or cache. If you are using a persistent working directory, you must reinitialize it whenever you change a configuration's providers
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
- The name given in the block header ("google" in this example) is the local name of the provider to configure. This provider should already be included in a `required_providers` block.
- There are also two "meta-arguments" that are defined by Terraform itself and available for all provider blocks:
  1. alias
     - for using the same provider with different configurations for different resources version, which we no longer recommend (use provider requirements instead) 
     ex: Multiple regions 
     - To declare a configuration alias within a module in order to receive an alternate provider configuration from the parent module, add the configuration_aliases argument to that provider's required_providers entry
  2. version
     - which we no longer recommend (use provider requirements instead)
- A `provider` block without an `alias` argument is the default configuration for that provider. Resources that don't set the provider meta-argument will use the default provider configuration that matches the first word of the resource type name
- When Terraform needs the name of a provider configuration, it expects a reference of the form `<PROVIDER NAME>.<ALIAS>`. These references are special expressions.  Like references to other named entities (for example, `var.image_id`), they aren't strings and don't need to be quoted. But they are only valid in specific meta-arguments of `resource`, `data`, and `module` blocks, and can't be used in arbitrary expressions.
- To use an alternate provider configuration for a resource or data source, set its provider meta-argument to a `<PROVIDER NAME>.<ALIAS>` reference
- To select alternate provider configurations for a child module, use its providers meta-argument to specify which provider configurations should be mapped to which local provider names inside the module
## Provider Requirements
- Each Terraform module must declare which providers it requires, so that Terraform can install and use them. Provider requirements are declared in a `required_providers` block.
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
- The `required_providers` block must be nested inside the top-level `terraform` block (which can also contain other settings).
- Each argument in the `required_providers` block enables one provider. The key determines the provider's local name.
- Each provider has two identifiers:
  1. A unique source address, which is only used when requiring a provider.
  2. A local name, which is used everywhere else in a Terraform module.
- Local names are module-specific, and are assigned when requiring a provider. Local names must be unique per-module
- A provider's source address is its global identifier. It also specifies the primary location where Terraform can download it. Source addresses consist of three parts delimited by slashes (/), as follows:
        `[<HOSTNAME>/]<NAMESPACE>/<TYPE>`   # Hostname (optional)
  ## Dependency Lock File
  - At present, the dependency lock file tracks only provider dependencies. Terraform does not remember version selections for remote modules, and so Terraform will always select the newest available module version that meets the specified version constraints. You can use an exact version constraint to ensure that Terraform will always select the same module version
  - The dependency lock file is a file that belongs to the configuration as a whole, rather than to each separate module in the configuration. For that reason Terraform creates it and expects to find it in your current working directory when you run Terraform, which is also the directory containing the .tf files for the root module of your configuration.
  - The lock file is always named `.terraform.lock.hcl`, and this name is intended to signify that it is a lock file for various items that Terraform caches in the `.terraform` subdirectory of your working directory.
  - Terraform automatically creates or updates the dependency lock file each time you run the `terraform init` command.
  - The dependency lock file uses the same low-level syntax as the main Terraform language, but the dependency lock file is not itself a Terraform language configuration file. It is named with the suffix `.hcl` instead of `.tf` in order to signify that difference.
  - If a particular provider already has a selection recorded in the lock file, Terraform will always re-select that version for installation, even if a newer version has become available. You can override that behavior by adding the `-upgrade` option when you run `terraform init`, in which case Terraform will disregard the existing selections and once again select the newest available version matching the version constraint.
