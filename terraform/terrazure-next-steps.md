# Notes: Terrazure next steps

Windows

* [x] Deploy Windows lab to Azure
* [ ] Make better use of variables
  *   [ ] Use Environment Variables for credentials

      * [ ] [https://learn.hashicorp.com/tutorials/terraform/sensitive-variables](https://learn.hashicorp.com/tutorials/terraform/sensitive-variables)
      *
      * [ ] Variable definition precedence - [https://www.terraform.io/docs/language/values/variables.html#variable-definition-precedence](https://www.terraform.io/docs/language/values/variables.html#variable-definition-precedence)
      *   [ ] Environment Variables - `export` and `printenv`&#x20;

          ```
          echo $[variable name]
          ```

      Positives of this method is no chance they are stored in files that could be subject to Git

      * [ ] Downside is you have to manage the state in your cmd line history
  *   [ ] .tfvars - can use secret variables `sensitive = true`

      * [ ] Don't forget gitignore - [https://github.com/github/gitignore/blob/master/Terraform.gitignore](https://github.com/github/gitignore/blob/master/Terraform.gitignore)
      * [ ] outputs can honour the sensitive label
      * [ ] When you run Terraform commands with a local state file, Terraform stores the state as plain text, including variable values, even if you have flagged them as `sensitive`. Since Terraform state can contain sensitive values, you must keep your state file secure to avoid exposing this data. -> [https://www.terraform.io/docs/language/state/sensitive-data.html](https://www.terraform.io/docs/language/state/sensitive-data.html) -> an argument for using remote state.&#x20;
      * [ ] Also consider a key vault


  * [ ] The `terraform.tfvars.json` file, if present.
  * [ ] Any `*.auto.tfvars` or `*.auto.tfvars.json` files, processed in lexical order of their filenames.
  * [ ] Any `-var` and `-var-file` options on the command line, in the order they are provided. (This includes variables set by a Terraform Cloud workspace.)
* [ ] WHAT about LOCALS
*   [ ] [https://www.terraform.io/docs/language/values/locals.html#when-to-use-local-values](https://www.terraform.io/docs/language/values/locals.html#when-to-use-local-values)

    * [ ] A local value assigns a name to an [expression](https://www.terraform.io/docs/language/expressions/index.html), so you can use it multiple times within a module without repeating it. Local values are like a function's temporary local variables.
    * [ ] Unlike input variables, locals are not set directly by users of your configuration.&#x20;
    * [ ] A good example is if you have to define required tags for resources such as environment and data classification. define them once in locals and they can't be overwritten by users (unlike variables) they can be specified in the module, or manipulating resource attributes eg. key\_vault\_name = split("/", azurerm\_key\_vault.tabwinkv.id)\[8]



    * [ ] Use tags to define security & compliance requirements e.g. data classification and Governance and regulatory compliance. \
      [https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json)
    *   [ ] default\_vm\_tags = {

        os\_family       = "windows"

        os\_distribution = lookup(var.vm\_image, "offer", "undefined")

        os\_version      = lookup(var.vm\_image, "sku", "undefined")

        }
    * [ ] ****[**Simplify Terraform Configuration with Locals**](https://learn.hashicorp.com/tutorials/terraform/locals)****
      * [ ] Techniques: You can add locals { name = repeating.variable } to the start of the main.tf
      * [ ] You can add multiple locals blocks
      * [ ] Combine locals with variables
        * [ ] start by defining your locals
        * [ ] add them to the variables
      * [ ] Use locals with values that you need control over
      * [ ] Use to simplify repetitive code
        * [ ] You can use it as part of _name = "vpc-${local.name\_suffix}"_
* [ ] What about DATA
  * [ ] _Data sources_ allow data to be fetched or computed for use elsewhere in Terraform configuration. Use of data sources allows a Terraform configuration to make use of information defined outside of Terraform, or defined by another separate Terraform configuration.
  * [ ] Good example with the public IP - [https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/data-sources/public\_ip](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/data-sources/public\_ip)
* [x] What happens to the keyvault itself? (recreated each time / unique name ) - **\`**
* [x] Comment out any unknown code # or /\* and \*/
* [x] Delete unused code/files



**ACTIONS:**

#### Tableau Script

Test out deploying again\
\
**Modules**

How to turn into tfvars variable?[https://registry.terraform.io/providers/hashicorp/external/latest/docs/data-sources/data\_source](https://registry.terraform.io/providers/hashicorp/external/latest/docs/data-sources/data\_source)\
[https://discuss.hashicorp.com/t/how-to-use-templatefile-to-pass-a-powershell-script-into-commandtoexecute/17916](https://discuss.hashicorp.com/t/how-to-use-templatefile-to-pass-a-powershell-script-into-commandtoexecute/17916)\
\
\
**Modules**

Should I rewrite to add those?

What about modules from the register?





