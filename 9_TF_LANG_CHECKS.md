# CHECKS
- The `check` block can validate your infrastructure outside the usual resource lifecycle. `check` blocks address a gap between post-apply and functional validation of infrastructure
- `check` blocks allow you to define `custom conditions` that execute on every Terraform plan or apply operation without affecting the overall status of an operation. `dheck` blocks execute as the last step of a plan or apply after Terraform has planned or provisioned your infrastructure
## Syntax
- You can declare a `check` block with a local name, zero-to-one scoped data sources, and one-to-many assertions.
- The following example loads the Terraform website and validates that it returns the expected status code 200.
```
check "health_check" {
  data "http" "terraform_io" {
    url = "https://www.terraform.io"
  }
  assert {
    condition = data.http.terraform_io.status_code == 200
    error_message = "${data.http.terraform_io.url} returned an unhealthy status code"
  }
}
```
- You can use any data source from any provider as a scoped data source within a `check` block
- A `check` block can optionally contain a nested (a.k.a. scoped) data source. This data block behaves like an external data source, except you can not reference it outside its enclosing `check` block
- Scoped data sources support the `depends_on` and `provider` meta-arguments. Scoped data sources do not support the `count` or `for_each` meta-arguments.
- The first time Terraform runs this check, it always throws a potentially distracting error message. You can fix this by adding `depends_on` to your scoped data source, ensuring it depends on an essential piece of your site's infrastructure. The check returns `known after apply` until that crucial piece of your website is ready
- Check blocks validate your custom assertions using `assert` blocks. Each `check` block must have at least one, but potentially many, `assert` blocks. Each `assert` block has a `condition` attribute and an `error_message` attribute.
- Check blocks offer the most flexible validation solution within Terraform. You can reference outputs, variables, resources, and data sources within `check` assertions. You can also use checks to model every alternate Custom Condition.
