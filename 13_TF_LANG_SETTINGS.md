# Terraform Settings
- The special `terraform` configuration block type is used to configure some behaviors of Terraform itself, such as requiring a minimum Terraform version to apply your configuration
- Terraform settings are gathered together into `terraform` blocks
- Each `terraform` block can contain a number of settings related to Terraform's behavior. Within a `terraform` block, only constant values can be used; arguments may not refer to named objects such as resources, input variables, etc, and may not use any of the Terraform language built-in functions
- The nested `cloud` block configures Terraform Cloud for enabling its CLI-driven run workflow.
- The nested `backend` block configures which state backend Terraform should use
- The `required_version` setting accepts a version constraint string, which specifies which versions of Terraform can be used with your configuration
- The `required_providers` block specifies all of the providers required by the current module, mapping each local provider name to a source address and a version constraint
- 
```
terraform {
  required_providers {
    aws = {
      version = ">= 2.7.0"
      source = "hashicorp/aws"
    }
  }
  cloud {}
  backend "http" {}
  
}
```
- The Terraform team will sometimes introduce new language features initially via an opt-in experiment, so that the community can try the new feature and give feedback on it prior to it becoming a backward-compatibility constraint
- In releases where experimental features are available, you can enable them on a per-module basis by setting the `experiments` argument inside a `terraform` block
```
terraform {
  experiments = [example]
}
```
- Depending on an experimental feature, any module with experiments enabled will generate a warning on every `terraform plan` or `terraform apply`. If you want to try experimental features in a shared module, we recommend enabling the experiment only in alpha or beta releases of the module
- The `terraform` block can have a nested `provider_meta` block for each provider a module is using, if the provider defines a schema for it. This allows the provider to receive module-specific information, and is primarily intended for modules distributed by the same vendor as the associated provider.
- Terraform Cloud
```
terraform {
  cloud {
    organization = "example_corp"
    ## Required for Terraform Enterprise; Defaults to app.terraform.io for Terraform Cloud
    hostname = "app.terraform.io"
    workspaces {
      tags = ["app"]
    }
  }
}
```
- You do not need to configure a backend when using Terraform Cloud because Terraform Cloud automatically manages state in the workspaces associated with your configuration. If your configuration includes a `cloud` block, it cannot include a `backend` block.
```
terraform {
  backend "remote" {
    organization = "example_corp"

    workspaces {
      name = "my-app-prod"
    }
  }
}
```
- You do not need to specify every required argument in the backend configuration. Omitting certain arguments may be desirable if some arguments are provided automatically by an automation script running Terraform. When some or all of the arguments are omitted, we call this a partial configuration
- There are several ways to supply the remaining arguments:
    1. File
        - To specify a file, use the `-backend-config=PATH` option when running `terraform init`. If the file contains secrets it may be kept in a secure data store, such as Vault, in which case it must be downloaded to the local disk before running Terraform
        - A backend configuration file has the contents of the backend block as top-level attributes, without the need to wrap it in another `terraform` or `backend` block
        ```
        address = "demo.consul.io"
        path    = "example_app/terraform_state"
        scheme  = "https"
        ```
        - `*.backendname.tfbackend` (e.g. `config.consul.tfbackend`) is the recommended naming pattern. Terraform will not prevent you from using other names but following this convention will help your editor understand the content and likely provide better editing experience as a result

    2. Command-line key/value pairs
        - Key/value pairs can be specified via the `init `command line. Note that many shells retain command-line flags in a history file, so this isn't recommended for secrets. To specify a single key/value pair, use the `-backend-config="KEY=VALUE"` option when running `terraform init`.
        ```
        terraform init \
            -backend-config="address=demo.consul.io" \
            -backend-config="path=example_app/terraform_state" \
            -backend-config="scheme=https"
        ```
    3. Interactively
        - Terraform will interactively ask you for the required values, unless interactive input is disabled. Terraform will not prompt for optional values

- Available Backends
    - `local`  : The local backend stores state on the local filesystem, locks that state using system APIs, and performs operations locally
    - `remote` : The remote backend is unique among all other Terraform backends because it can both store state snapshots and execute operations for Terraform Cloud's CLI-driven run workflow. It used to be called an "enhanced" backend.
    - `azurerm`: Stores the state as a Blob with the given Key within the Blob Container within the Blob Storage Account.
    - `consul` : Stores the state in the Consul KV store at a given path
    - `cos`    : Stores the state as an object in a configurable prefix in a given bucket on Tencent Cloud Object Storage (COS).
    - `gcs`    : Stores the state as an object in a configurable prefix in a pre-existing bucket on Google Cloud Storage (GCS). The bucket must exist prior to configuring the backend
    - `http`   : Stores the state using a simple REST client.
    - `kubernetes` : Stores the state in a Kubernetes secret.
    - `oss`    : Stores the state as a given key in a given bucket on Stores Alibaba Cloud OSS. This backend also supports state locking and consistency checking via Alibaba Cloud Table Store, which can be enabled by setting the `tablestore_table` field to an existing TableStore table name.
    - `pg`     : Stores the state in a Postgres database version 10 or newer
    - `s3`     : Stores the state as a given key in a given bucket on Amazon S3. This backend also supports state locking and consistency checking via `Dynamo DB`, which can be enabled by setting the `dynamodb_table` field to an existing DynamoDB table name. A single DynamoDB table can be used to lock multiple remote state files. Terraform generates key names that include the values of the bucket and key variables

ex: Datasource configuration
```
    data "terraform_remote_state" "foo" {
        backend = "azurerm"
        config = {
            storage_account_name = "terraform123abc"
            container_name       = "terraform-state"
            key                  = "prod.terraform.tfstate"
        }
    }
```
