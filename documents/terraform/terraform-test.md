# Terraform-Test

The UDS Common Workflow for Terraform has agreed to use [Terratest](https://terratest.gruntwork.io/). This testing framework does require the use of and limited knowledge of the [Go programming language](https://go.dev/). The experience needed is not extensive as there should be plenty of examples using Terratest in the Defense Unicorns Terraform repositories.

For this GitHub Workflow to be utilized properly, a common structure and pattern has been implemented to make these workflows reusable. The goal is to make it as easy as possible to deliver tested UDS packages.

*Overall Objective:* Provide testing at the repository level. Testing should include a base case and additional tests will be added as needed for common behaviors relied upon downstream from the repository when deemed necessary upon request and review by the team.

## Workflow Arguments

* test_retry:
  * required: false
  * description: 'Retry count'
  * default: 1
  * type: number

*Example found in this [section](#example-test).*

## Credentials

The team has decided that credentials must be defined at the repository level and not in the shared workflow. This will allow the development team to test in the environment that they need specifically without the common workflow mandating which repository and secrets must be used.

~~To use, make sure when the shared workflow is used that the shared workflow is called with the `secrets: inherit` flag (show in the [example test](#example_tst) section). GitHub has documented [Simplifying using secrets with resuable workflows](https://github.blog/changelog/2022-05-03-github-actions-simplify-using-secrets-with-reusable-workflows/) on how and why.~~

It has been found that `secrets: inherit` is not needed when using a shared action instead of a shared workflow. The reason for using an action is due to the host repository needing to supply the credentials. Workflows cannot call a workflow after calling an action, but a workflow may call multiple actions.

### AWS Credentials

To set the AWS credentials, one must have:

* Access to the CI account.
* Find or know the role associated with the GitHub Actions access.
* Write permissions to the role's trust relationship.

**Steps to Connect to AWS:**

1. Connect to the AWS account used for CI or whatever account that will used.
2. Find the role associated with any GitHub CI/CD work.
3. Copy the role arn and set as a repository secret in the GitHub Repository.
   1. Click on the repository **Settings** --> **Secrets and Variables** --> **Actions** --> **New repository secret** button.
   2. Name like in the example or something this obvious.
   3. Copy contents in the window.
4. Go back to the AWS role in the web console, like on the **Trust relationships** tab.
5. Find the block of related repositories, add the new repository to the list without breaking the JSON format.
6. Test the workflow in GitHub, the AWS Credentials action will fail if there is a misconfiguration.

Additional [AWS Tips blog post](https://awstip.com/using-github-actions-oidc-to-run-terraform-in-aws-31ba395518cb) explaining the steps above in-depth and AWS initialization steps.

## Workflow Permissions

Through experimentation, the following permissions block is required to use the repository secret and the `configure-aws-credentials` action.

```yaml
permissions:
  id-token: write
  contents: read
```

If the secret has been properly set, repository added to the trust relationship of the role in the account, and still fails, the permissions of the workflow is often overlooked (see [example](#example-test)).

## Repository Structure

The layout required for testing uniformly are specified as followed:

* All Go source files must be in a `test/` folder.
* All Go source code must use the `test_test` module name.
  * Reason: `test` namespace is reserved and a warning will be shown.
* The `go.mod` must reside in the root of the directory.
  * Necessary as the command `go test -count <count> -v ./...` will be used.
  * The `go.mod` file must be initialized with `go mod init test_test` while in the repositry root directory.
* An `examples/` folder must be created to house the testing terraform of the module.

## Example Test

Given the knowledge that not every Terraform module is the same in terms of complexity and the complexity of the tests may range from seconds to test to almost an hour. The minimum suggestiong is that all Terraform be able to be tested before a Pull Request is approved and shown in GitHub.

Please research what is best for the specific repository, but an easy target for a baseline for testing is with the `pull_request` event. The reader has flexibility for more specific events in `pull_request` as needed depending the scope of the Terraform to be tested.

**Installation:** The following can be copied and pasted into `.github/workflows/terraform-test.yaml`. Assuming the permissions, secret, AWS role trust relationship, and structure of the Go files/Terraform examples in the right place; the example should just work.

```yaml
name: Terraform Test
on:
  pull_request:

permissions:
  id-token: write
  contents: read

defaults:
  run:
    # We need -e -o pipefail for consistency with GitHub Actions' default behavior
    shell: bash -euo pipefail

jobs:
  terraform-test:
    runs-on: ubuntu-latest
    if: github.event.pull_request.draft == false
    steps:
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: ${{ secrets.AWS_COMMERCIAL_ROLE_TO_ASSUME }}
          role-session-name: ${{ github.event.client_payload.pull_request.head.sha || github.sha }}
          aws-region: us-east-1
          # 21600 seconds == 6 hours
          # 1800 seconds == 30 minutes
          role-duration-seconds: 1800

      - name: Run Shared Test workfow
        uses: defenseunicorns/uds-common-workflows/.github/actions/terraform-test@main
        with:
          test_retry: 1
```

## Workflow Diagram

For the workflow to complete, all steps must complete successfully.

```mermaid
flowchart LR
  subgraph "Local Terraform Worklow"
    trigger[GitHub Pull Request Event] --> aws[AWS Credentials Action]
    aws --> clone[GitHub Clone Action]

    subgraph "Terraform Test Shared Action"
      clone --> goset[Install Go]
      goset --> tidy(Install Go dependencies)
      tidy --> test(Go Test)
    end
    
    test --> endflow[End]
  end
```
