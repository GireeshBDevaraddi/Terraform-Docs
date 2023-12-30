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
- A `data` block requests that Terraform read from a given data source ("aws_ami") and export the result under the given local name ("example"). The name is used to refer to this resource from elsewhere in the same Terraform module, but has no significance outside of the scope of a module
- The data source and name together serve as an identifier for a given resource and so must be unique within a module.
- When distinguishing from data resources, the primary kind of resource (as declared by a resource block) is known as a managed resource. Both kinds of resources take arguments and export attributes for use in configuration, but while managed resources cause Terraform to create, update, and delete infrastructure objects, data resources cause Terraform only to read objects
  - Most of the items within the body of a `data` block are defined by and specific to the selected data source, and these arguments can make full use of expressions and other dynamic Terraform language features
  - Terraform reads data resources during the planning phase when possible but announces in the plan when it must defer reading resources until the apply phase to preserve the order of operations
  - Terraform defers reading data resources in the following situations:
    1. At least one of the given arguments is a managed resource attribute or other value that Terraform cannot predict until the apply step.
    2. The data resource depends directly on a managed resource that itself has planned changes in the current plan.
    3. The data resource has custom conditions and it depends directly or indirectly on a managed resource that itself has planned changes in the current plan

- While many data sources correspond to an infrastructure object type that is accessed via a remote network API, some specialized data sources operate only within Terraform itself, calculating some results and exposing them for use elsewhere. For example, local-only data sources exist for rendering templates and reading local files.
- The behavior of `local-only` data sources is the same as all other data sources, but their result data exists only temporarily during a Terraform operation, and is re-calculated each time a new plan is created.
- Due to the data resource behavior of deferring the read until the apply phase when depending on values that are not yet known, using depends_on with data resources will force the read to always be deferred to the apply phase, and therefore a configuration that uses depends_on with a data resource can never converge. Due to this behavior, we do not recommend using depends_on with data resources
- You can use `precondition` and `postcondition` blocks to specify assumptions and guarantees about how the data source operates
- Data resources support `count` and `for_each` meta-arguments as defined for managed resources, with the same syntax and behavior.
- As with managed resources, when `count` or `for_each` is present it is important to distinguish the resource itself from the multiple resource instances it creates. Each instance will separately read from its data source with its own variant of the constraint arguments, producing an indexed result
- Data resources support the `provider` meta-argument as defined for managed resources, with the same syntax and behavior.
- Data resources do not have any customization settings available for their lifecycle. However, the `lifecycle` block is reserved for future versions.
- Each data instance will export one or more attributes, which can be used in other resources as reference expressions of the form `data.<TYPE>.<NAME>.<ATTRIBUTE>`

## Data Source Lifecycle
- If the arguments of a data instance contain no references to computed values, such as attributes of resources that have not yet been created, then the data instance will be read and its state updated during Terraform's "refresh" phase, which by default runs prior to creating a plan. This ensures that the retrieved data is available for use during planning and the diff will show the real values obtained.
