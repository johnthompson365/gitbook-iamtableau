# Terrazure next steps

Windows

* [x] Deploy Windows lab to Azure
* [ ] Make better use of variables
  * [ ] Use Environment Variables for credentials

    * [ ] [https://learn.hashicorp.com/tutorials/terraform/sensitive-variables](https://learn.hashicorp.com/tutorials/terraform/sensitive-variables)
    * [ ] * [ ] Variable definition precedence - [https://www.terraform.io/docs/language/values/variables.html\#variable-definition-precedence](https://www.terraform.io/docs/language/values/variables.html#variable-definition-precedence)
    * [ ] Environment Variables - `export` and `printenv` 

      ```text
      echo $[variable name]
      ```

    Positives of this method is no chance they are stored in files that could be subject to Git

    * [ ] Downside is you have to manage the state in your cmd line history

  * [ ] .tfvars - can use secret variables `sensitive = true`

    * [ ] Don't forget gitignore - [https://github.com/github/gitignore/blob/master/Terraform.gitignore](https://github.com/github/gitignore/blob/master/Terraform.gitignore)
    * [ ] outputs can honour the sensitive label
    * [ ] When you run Terraform commands with a local state file, Terraform stores the state as plain text, including variable values, even if you have flagged them as `sensitive`. Since Terraform state can contain sensitive values, you must keep your state file secure to avoid exposing this data. -&gt; [https://www.terraform.io/docs/language/state/sensitive-data.html](https://www.terraform.io/docs/language/state/sensitive-data.html) -&gt; an argument for using remote state. 
    * [ ] Also consider a key vault

  * [ ] The `terraform.tfvars.json` file, if present.
  * [ ] Any `*.auto.tfvars` or `*.auto.tfvars.json` files, processed in lexical order of their filenames.
  * [ ] Any `-var` and `-var-file` options on the command line, in the order they are provided. \(This includes variables set by a Terraform Cloud workspace.\)
* [ ] WHAT about LOCALS
  * [ ] A local value assigns a name to an [expression](https://www.terraform.io/docs/language/expressions/index.html), so you can use it multiple times within a module without repeating it. Local values are like a function's temporary local variables.
  * [ ] a good example is if you have to define tags for resources define them once in locals and specify in the module, or manipulating resource attributes eg. key\_vault\_name = split\("/", azurerm\_key\_vault.tabwinkv.id\)\[8\]
  * [ ] default\_vm\_tags = {

    os\_family       = "windows"

    os\_distribution = lookup\(var.vm\_image, "offer", "undefined"\)

    os\_version      = lookup\(var.vm\_image, "sku", "undefined"\)

    }

  * [ ] \*\*\*\*[**Simplify Terraform Configuration with Locals**](https://learn.hashicorp.com/tutorials/terraform/locals)\*\*\*\*
* [ ] What about DATA
  * [ ] _Data sources_ allow data to be fetched or computed for use elsewhere in Terraform configuration. Use of data sources allows a Terraform configuration to make use of information defined outside of Terraform, or defined by another separate Terraform configuration.
  * [ ] Good example with the public IP - [https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/data-sources/public\_ip](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/data-sources/public_ip)
* [x] What happens to the keyvault itself? \(recreated each time / unique name \) - **\`**
* [x] Comment out any unknown code \# or /\* and \*/
* [x] Delete unused code/files



ACTIONS:

#### TAGS

Use tags to define security & compliance requirements e.g. data classification and Governance and regulatory compliance.   
[https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json)  
Look at using Locals or Variables for this. Consider Windows and Linux.  
resource "azurerm\_resource\_group"

variable "tags" { type = map

default = { Environment = "Tableau-Windows" } }

#### OUTPUTS

Test outputs by deploying server.

#### DATA

AFAIK this is the only place it is used. It is required -&gt; 

output "public\_ip\_address" { value = data.azurerm\_public\_ip.ip.ip\_address }

[https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/data-sources/public\_ip](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/data-sources/public_ip)







