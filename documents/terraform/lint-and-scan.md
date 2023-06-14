# Lint and Scan

This article will explain how the `.github/workflows/terraform-scan.yaml` operates and how to use the shared workflow. Please note, Terraform linting and security scanning are performed in a single workflow.

## Contents of the Workflow

* **GitHub Actions Workflow Type:** Call
* **Variables:**
  * *soft-fail* 
    * Used by Tfsec. When set to true, vulnerabilities found will not stop the pipeline but still show a report.
  	* required: false
  	* default: false
  	* type: boolean
  * *tf-dirs*
    * Used by Regula, this variable can either be line (heredoc) or space seperated for multiple folders.
    * required: false
    * default: .
    * type:string

### [Tflint](https://github.com/terraform-linters/setup-tflint)

This is a Terraform linter that will do a very general look at any files with the `.tf` suffix. The action will report any linting that seems out place with regards to `type` not set properly for a variable for example. This linter will not correct anything for you and will not worry about the format of the Terraform.

The settings for this action are to have compact results and to recursively scan subfolders for any Terraform files included as well.

### [Tfsec](https://github.com/aquasecurity/tfsec-action)

Tfsec is a well known and established Terraform security scanning tool. The parent company is Aqua Security and has been around the industry a while as a leader. The reports simple to read at the end and runs fairly quickly.

### [Regula](https://github.com/fugue/regula-action)

Regula is another impressive scanner that, when configured, can apply additional rules for scanning. Regula must be told what folders to scan (defaul is the repo root folder). Please note, regula is working even though the results may lead to improper assumptions. Regula's stock enforcement will mostly look at known providers and provide scans. Regula seems to be one of the easiest tools to optimize to add or dismiss certain rules when scanning Terraform.

## Workflow Diagram

For the worklow to work, all jobs must bass for a successful pipeline.

```mermaid
flowchart LR

  subgraph "Terraform Scan Workflow"
    Start --> tflint(Tflint)
    tflint --> tfsec(Tfsec)
    tflint --> regula(Regula)
  end
```