# TERRAFORM TESTS
- Terraform tests let authors validate that module configuration updates do not introduce breaking changes. Tests run against test-specific, short-lived resources, preventing any risk to your existing infrastructure or state.
- Each Terraform test lives in a test file. Terraform discovers test files are based on their file extension: `.tftest.hcl` or `.tftest.json`
- Each test file contains the following root level attributes and blocks:
    1. One to many `run` blocks.
    2. Zero to one `variables` block.
    3. Zero to many `provider` blocks
- You can override the normal testing behavior by updating the `command` attribute within a `run` block (examples below). By default, each `run` block executes with `command = apply` instructing Terraform to execute a complete apply operation against your configuration. Replacing the command value with `command = plan` instructs Terraform to not create new infrastructure for this `run` block. This allows test authors to validate logical operations and custom conditions within their infrastructure in a process analogous to unit testing
- Terraform executes `run` blocks in order, simulating a series of Terraform commands executing directly within the configuration directory. The order of the `variables` and `provider` blocks doesn't matter, Terraform processes all the values within these blocks at the beginning of the test operation. We recommend defining your `variables` and `provider` blocks first, at the beginning of the test file.
- Each `run` block has the following fields and blocks:
    |Field or Block Name	     | Description   	                                             | Default Value |
    |---------------------------|----------------------------------------------------------------|---------------|
    |command	                | An optional attribute, which is either apply or plan.	         | apply         |
    |plan_options.mode	        | An optional attribute, which is either normal or refresh-only. | normal        |
    |plan_options.refresh	    | An optional boolean attribute.	                             | true          |
    |plan_options.replace	    | An optional attribute containing a list of resource addresses  |               |
    |                           |referencing resources within the configuration   under test.	 |               |
    |plan_options.target	    | An optional attribute containing a list of resource addresses  |               |
    |                           | referencing resources within the configuration under test.	 |               |
    |variables	                | An optional variables block.	                                 |               |
    |module	                    | An optional module block.	                                     |               |
    |providers	                | An optional providers attribute.	                             |               |
    |assert	                    | Optional assert blocks.	                                     |               |
    |expect_failures	        | An optional attribute.	                                     |               |
- Terraform run block `assertions` are Custom Conditions, consisting of a `condition` and an `error message`.
- The test file syntax supports `variables` blocks at both the root level and within `run` blocks. Terraform passes all variable values from the test file into all `run` blocks within the file. You can override variable values for a particular `run` block with values provided directly within that `run` block
- In addition to specifying variable values via test files, the `Terraform test` command also supports the other typical mechanisms for specifying variable values.
- You can set or override the required providers within the main configuration from your testing files by using `provider` and `providers` blocks and attributes.
- You can modify the module that a given `run` block executes
- By default, Terraform executes the given command against the configuration being tested for each run block. Terraform tests the configuration within the directory you execute the `terraform test` command from (or the directory you point to with the `-chdir` argument). Each `run` block also allows the user to change the targeted configuration using the module block.
- Unlike the traditional `module` block, the `module` block within test files only supports the source attribute and the version attribute. The remaining attributes that are typically supplied via the traditional module block should be supplied by the alternate attributes and blocks within the `run` block
- While Terraform executes a `terraform test` command, Terraform maintains at least one, but possibly many, state files within memory for each test file.
- Terraform destroys resources in the following order, and this order is important because it affects the structure of your testing files:
    1. Resources in the main state file. Do not create resources in alternate modules that depend on resources from your main configuration.
        - Data sources can refer to objects in your main configuration, because Terraform does not have to destroy data sources.
    2. Resources created by alternate modules in reverse run block order.
- By default, if any Custom Conditions, including `check` block assertions, fail during the execution of a Terraform test file then the overall command reports the test as a failure. However, it is a common testing paradigm to want to test failure cases. Terraform supports the `expect_failures` attribute for this use case.
- In each `run` block the `expect_failures` attribute can provide a list of checkable objects (resources, data sources, check blocks, input variables, and outputs) that should fail their custom conditions. The test passes if the checkable objects you specify report an issue, and the test fails overall if they do not

```
# customised_provider.tftest.hcl

provider "aws" {
    region = "eu-central-1"
}

variables {
  bucket_prefix = "test"
}

run "valid_string_concat" {

  command = plan

  assert {
    condition     = aws_s3_bucket.bucket.bucket == "test-bucket"
    error_message = "S3 bucket name did not match expected"
  }

}
```
