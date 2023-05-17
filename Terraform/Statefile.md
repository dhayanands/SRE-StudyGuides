# Terraform State file

## State File

- Is in JSON format
- Stores the mapping data of resource identifier in Terraform configuration and the unique identifier of deployment
- The state file is refeshed by queriying the deployment and updating the identifier data
- Also stores metadata infos :
  - version of Terraform
  - version of State data format
  - serial number of current state data
- Must not be modified manually, use terraform command to modify
- During Terraform operations, the state data is locked so it cannot be modified by any other Terraform instances
- Location of statefile: can be local on disk or on remote server / storage
- `Workspaces` allow us to create multiple instances of deployment with seperate state files using the same configuration

```bash
{
  "version": 4, # version of state data format
  "terraform_version": "0.12.31", # version of Terraform last used on the data
  "serial": 63, # incremented each time the state data is updated
  "lineage": "07f365af-24db-324b-f298-0228f0868d60",  # unique id associated with each instance of state data
  "outputs": {}, # output data printed in the last run
  "resources": [] # list of resource mapping and attributes
```

## State data commands

`terraform state list`  - list all state resources in state
`terraform state show <ADDRESS>`  - get info about specific resource in state, ADDRESS is resource_type=resource_name
`terraform state mv <SOURCE> <DESTINATION>` - move or rename a resource in sate
`terraform state rm <ADDRESS>`  - delete a resource in state


