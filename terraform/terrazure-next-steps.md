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
* [ ] What happens to the keyvault itself? \(recreated each time / unique name \)
* [ ] Comment out any unknown code \# or /\* and \*/
* [ ] Delete unused code/files

