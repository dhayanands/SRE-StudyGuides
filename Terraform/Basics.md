# Terraform Basics

- [Terraform Basics](#terraform-basics)
  - [Core Components](#core-components)
  - [Terraform Object Types](#terraform-object-types)
  - [General Block of Syntax](#general-block-of-syntax)
  - [Terraform Workflow](#terraform-workflow)

## Core Components

- Executable
  - single binary file that is invoked to run Terraform
  - contains all the core Terraform funtionality

- Configuration File
  - All configurations to be deployed with extension `.tf`
  - Maybe in one or more files

- Provider Plugin
  - Used to interact with the various services like AWS, Azure, OCI, etc.,
  - These are executables invoked by Terraform to interact with a services's API
  - Most common ones are hosted at public Terraform registry `registry.terraform.io`

- State Data
  - Used to keep track of the state of the configuration once its deployed
  - Map of what is defined in the configuration and what exists in the target environment
  - When we need to update the environment, Terraform compares the updated configuration to the state file, calulates the changes that need to be made to match both and then updates the sate

## Terraform Object Types

- Providers
  - defines information about the provider we want to use - eg: AWS, Azure etc.,
- Resources
  - things that we want to create in the target environment
  - each resource is associated with a provider
  - eg: VM, database, load balancer, storage etc.,
- Datasources
  - a way to query information from a provider
  - datasources are also associated with the providers
  - eg: list of availability zones, list of templates for a VM etc.,

## General Block of Syntax

- HashiCorp Configuration Language uses block syntax for everything in file
- a simplified version of json - easier to read & supports inline comments

```bash
// each block starts with a `block_type` keyword that descibe what type of object is being described in the block
// the `block_type` keyword is followed by labels depending on the type of object being created
// and the last label is the `name_label` that refers to the object being defined
block_type "label" "name_label" {
    
    // one or more key-value pairs that makes use of the availble arguments for the objecvt type being defined
    // each key will be a string and the value can be any data type that the augument requires 
    key = "value"

    // we can also have nested block with more key-value pairs
    nested_block {
        key = "value"
    }
}
```

**example:**

an example block to create a webserver in AWS with stotage of 40Gb

```bash
resource "aws_instance" "web_server" {
    name = "web_server"
    ebs_volume {
        size = 40
    }
}
```

**Terraform Object Reference:**

`<resource_type>.<name_label>.<attribute>`

eg: `aws_instance.web_server.name`

## Terraform Workflow

`terraform init`

- looks for configuration files in the current working directory and examines to see if they need any provider plugin
- if a plugin is needed, Terraform downloads the plugin from the public plugin registry
- part of the init process is also to get the state data backend ready. if no backend is specified, a ste file is created in the current working directory
- once initialization is complete, Terraform is ready to deploy infrastructure

`terraform plan`

- Terraform will look at the current configuration and also content of the state data, determine the differences between the two and make a plan to update the target environment to match the desired configuration
- Terraform will printout the plan for us to look and verify the changes that will be made
- We can save the plan changes to a file and feed that back to Terraform to make the actual changes later

`terraform apply`

- The Terraform plan changes are given as input during this step and Terraform uses the provider plugins to make the actual changes
- Based on the plan, the resources will be created or modified on the target environment and the state data will be updated to reflect the changes

`terraform destroy`

- destroy / delete everything in the target environment based on the state data

`terraform validate`

- To validate the syntax and logic of the configuration
- Before running the validation, `terraform init` should be run to initiate the provider plugins
- The validate command does not check the state of the deployment
- It is no guarantee that the deployment will be successful if the validation is successful
