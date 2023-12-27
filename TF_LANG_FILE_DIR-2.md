# FILES AND DIRECTORIES
- Code in the Terraform language is stored in plain text files with the .tf file extension. There is also a JSON-based variant of the language that is named with the .tf.json file extension. Files containing Terraform code are often called configuration files
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
  - `resource` and `data` Block override
    - Within a resource block, the contents of any lifecycle nested block are merged on an argument-by-argument basis
    - If an overriding resource block contains one or more provisioner blocks then any provisioner blocks in the original block are ignored.
    - If an overriding resource block contains a connection block then it completely overrides any connection block present in the original block.
    - The depends_on meta-argument may not be used in override blocks, and will produce an error
  - `variable` block override
    - The arguments within a variable block are merged in the standard way 
    - some special considerations apply due to the interactions between the type and default arguments.If the original block defines a default value and an override block changes the variable's type, Terraform attempts to convert the default value to the overridden type, producing an error if this conversion is not possible.
    - Conversely, if the original block defines a type and an override block changes the default, the overridden default value must be compatible with the original type specification
  - `output` Block override
    - The depends_on meta-argument may not be used in override blocks, and will produce an error.
  - `locals` Block override
    - Each locals block defines a number of named values. Overrides are applied on a value-by-value basis, ignoring which locals block they are defined in.
  - `terraform` block override 
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
  - The dependency lock file is a file that belongs to the configuration as a whole, rather than to each separate module in the configuration. For that reason, Terraform creates it and expects to find it in your current working directory when you run Terraform, which is also the directory containing the .tf files for the root module of your configuration
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
