# drift-detection
Action for testing if there are changes in IDP structured Tofu (Terraform) code

Nothing in this action will ever attempt to apply changes to your infrastructure.

Given a folder structure where the repository contains 
`/infrastructure/accounts/<<account-name>>/<<account-id>>/<<numbered-groups>>``
this action can traverse the structure and detect changes not reflected in the 
infrastructure code as well as invalid Tofu syntax and other Tofu validation errors. 

## Example folder structure
In this example the repository contains infrastructure for the accounts 
`aws-ai-test`, `aws-koa-dev` and `jppol-idp-test`. They contain various infrastructure 
folders each. The structure with the numbered directories supports applying elements 
with dependencies, as everyhing in layer 01 will be applied after layer 00 but before 
layer 02.  Multiple folders with same numbers can exist. 

```

└── infrastructure
    ├── accounts
    │   ├── aws-ai-test
    │   │   └── 058264145527
    │   │       ├── 00.bootstrap
    │   │       ├── 01.github-repos
    │   │       ├── 02.eks
    │   │       └── 03.vpc-peering
    │   ├── aws-koa-dev
    │   │   └── 931827427725
    │   │       ├── 00.bootstrap
    │   │       ├── 01.github-repos
    │   │       ├── 01.secrets-manager
    │   │       ├── 02.eks
    │   │       └── 03.vpc-peering
    │   └── jppol-idp-test
    │       └── 971422674709
    │           ├── 00.bootstrap
    │           ├── 01.github-repos
    │           ├── 01.secrets-manager
    │           └── 02.eks
    └── modules
        ├── child-access-to-state
        ├── secrets-access
        └── state
```
## Default branched - Connection to branches
The action assumes contents of each folder should match `main` - if an account should 
be connected to a different standard branch it is possible to put a text file `branch.txt`
containing a single line with 

## Running the action
The action can be executed for both pushes, when PRs exist, as a scheduled task and using 
workflow dispatch (running it manually from github.com). 

Behavior will differ: When running as scheduled task, changes will be compared to the repository 
in the default branch for the specific account. When run as part of a PR or a workflow dispatch, 
it will simply show the dirrence between the current branch and the actual state of the account.

## Output 
When running as part of a PR, the detected Tofu plan will be posted to the PR conversation. 

When changes are run as part of a scheduled run, changes will can be posted to Slack. 

## Behavior on failure 
When drift detect fails for any directory due to Tofu errors, a message can be posted to Slack. 


# Connecting to Slack 
## How to configure Slack 

## How to configure repository
Drift detect will be reported to slack using a webhook url named `IDP\_SLACK\_REPORT\_DRIFT\_URLì`. 
Drift is only reported to Slack as part of scheduled run, as it is intended to alert if your infrastructure
does not match "expected" structure. Comparison happens against the accounts standard branch. 

Errors will reported using `IDP\_SLACK\_REPORT\_ERROR\_URL`. All syntax errors are reported regardless of 
the run conditions. 

Setting a blank or no value for any of the two mentioned secrets will suppress attempts to report via Slack. 

# Part of validation flows
If a plan can be created for all account folders the workload `drift-detect-completed` is executed. 

As this part of the action is only started if all folders validates ok, it can be integrated in branch rules 
for pull requests etc. 
