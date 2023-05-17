# Adding Resources

## Workflow

- Create a datasource to get all information from the cloud provider (availability zones, LB / VM types etc.,)
- List all the resources to be created
  - EXAMPLE: to create server with LB we will need
    - Network setup
    - Instance / VM that runs the application
    - Load Balancer
    - Target group for load Balancer
    - Listener configuration for target group
    - Attach it to the VM
  - The resource group and resource setup might vary slightly between cloud providers
- Create one configuration file for each of the group like:
  - network.tf - for configurations related to IP address
  - loadbalancer.tf - for configurations related to Load Balancer
  - instances.tf - for configurations related to VM instances

**Basic structure of a configuration file:**

```json
// ABC.tf

##################################################################################
# PROVIDERS
##################################################################################

// only for main.tf - ALL THE PROVIDER DEFINITION GO HERE

##################################################################################
# DATA
##################################################################################

// ALL DATASOURCE INFO GO HERE

##################################################################################
# RESOURCES
##################################################################################

// ALL RESOURCE DEFINITIONS GO HERE
```
