# Terraform-Test

The UDS Commown Workflow for Terraform has agreed to use [Terratest](https://terratest.gruntwork.io/). This testing framework does require the use of and limited knowledge of the [Go programming language](https://go.dev/). The experience needed is not extensive as there should be plenty of examples using Terratest in the Defense Unicorns Terraform repositories. 

For this GitHub Workflow to be utilized properly, a common structure and pattern has been implemented for ensuring this workflow can be reuseable for all related repositories.  Testing is a mandatory feature of all UDS packages and these shared workflows are supplied to ease the use of future development that meets the defined standards of a UDS package.

*Overal Objective:* Provide testing at the repository level. Testing should include a base case and additional tests will be added as needed for common behaviors relied upon downstream from the repository when deemed necessary upon request and review by the team. 

## Workflow Arguments 

* test_retry:
  * required: false
  * description: 'Retry count'
  * default: 3
  * type: number

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

```yaml
name: Terraform Test
on:
  pull_request:

jobs:
  terraform-test:
    uses: defenseunicorns/uds-common-workflows/.github/workflows/terraform-test.yml@25-common-iac-test
    with:
      test_retry: 1
```

## Workflow Diagram

```mermaid
flowchart TB
  subgraph "Local Terraform Worklow"
    trigger[GitHub Pull Request Event]
  end

  subgraph "Terraform Test Shared Workflow"
	trigger --> clone[GitHub Clone Action]
	clone --> tidy(Install Go dependencies)
	tidy --> test(Go Test)
	test --> endflow[End]
  end
```
