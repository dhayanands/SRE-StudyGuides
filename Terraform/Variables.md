# Data in Terraform

- [Data in Terraform](#data-in-terraform)
  - [Variable Types](#variable-types)
  - [Variable Data Types](#variable-data-types)
  - [Variable](#variable)
    - [Variable Definition](#variable-definition)
    - [Variable Reference](#variable-reference)
    - [Reference Variable in variables.tf](#reference-variable-in-variablestf)
  - [Local Variable](#local-variable)
    - [Local Variable Definition in locals.tf](#local-variable-definition-in-localstf)
    - [Local Variable Reference](#local-variable-reference)
  - [Output Variable](#output-variable)
    - [Output Variable Definition](#output-variable-definition)
  - [Supply Value to Variable](#supply-value-to-variable)

## Variable Types

- **Input Variables** are variables that are passed to Terraform and defined as part of configuration
- **Local Values** are computed values inside configuration. These are not passed directly to Terraform as an external input or defined in configuration. They are computed based on the input variables and internal references
- **Output values** are returned from Terraform. Outputs are defined in configuration and the vaue will depend on what it references

## Variable Data Types

- 3 Types : Primitive, Collection and  Structural

- Primitive
  - string - sequence of unicode characters
  - number - integer or decimal
  - boolean - true or false

- Collection - group of Primitive data types, the vaules stored should be of same data type
  - list - ordered group of elements
  - set - unordered list of elements
  - map - key-value pairs

- Structural - group of Primitive data types, the vaules stored can be of fifferent data type
  - tuple - equivalent to list
  - objects - equivalent to maps

**EXAMPLES:**

```bash
## LIST
[1, 2, 3, 4]
["us-east-1","us-east-2","us-west-1","us-west-2"]
[1,"us-east-2","us-west-1",true]    # invalid list

## MAP
{
    small = "t2.micro"
    medium = "t2.small"
    large = "t2.large"
}
```

## Variable

### Variable Definition

```bash
// main.tf

// variable with no arguments
variable "name_label" {}

// variable with arguments
variable "name_label" {
    type = value                // data type of the variable
    description = "value"       // provide context for the user, useful in case of errors
    default = "value"           // provide default value to variable, if not provided by user. Or Terraform will prompt for input
    sensitive = true | false    // if set to true, Terraform will not show the value in output or logs
}
```

### Variable Reference

- variables are referenced using the `vars` keyword

```bash
## refer an element
var.<name_label>

var.aws_region

## refer a list element
vars.<name_label>.[<elemet_number>]

var.aws_regions[0]


## refer a map element
vars.<name_label>.<key_name>
vars.<name_label>.[key_name]

var.aws_instance_size.small
var.aws_instance_size.["small"]
```

### Reference Variable in variables.tf

- The variables can be seperated form the `main.tf` file and all variables can be kept in `variables.tf`
- Once defined in variables.tf file, they can be referenced as usual with the `var` keyword in other files as Terraform processes all `.tf` files together

**EXAMPLE:**

- define variables in variables.tf

```bash
// variables.tf

variable "vpc_cidr_block" {
  type        = string
  description = "Base CIDR Block for VPC"
  default     = "10.0.0.0/16"
}

variable "vpc_subnets_cidr_block" {
  type        = list(string)
  description = "CIDR Blocks for subnets in VPC"
  default     = ["10.0.0.0/24","10.0.1.0/24","10.0.2.0/24","10.0.3.0/24"]
}

variable "enable_dns_hostnames" {
  type        = bool
  description = "Enable DNS hostnames in VPC"
  default     = true
}
```

- reference the defined variables in main.tf

```bash
// main.tf
# NETWORKING #
resource "aws_vpc" "vpc" {
  cidr_block           = var.vpc_cidr_block
  enable_dns_hostnames = var.enable_dns_hostnames
}
```

## Local Variable

- All the local variables can be kept in `locals.tf`
- Common usecase will be to construct `tags` that needs to be added to resources

### Local Variable Definition in locals.tf

```bash
// starting a local block in locals.tf

locals{
  // define the common tags that will be used by all resources
  common_tags ={
    // var.company, var.projects  var.billing_code are variables defined in variables.tf
    company = var.company
    project = "${var.company}-{var.project}"
    billing_code = var.billing_code
  }
}
```

### Local Variable Reference

- Local variables are referenced using the `local` keyword

```bash
// based on common_tags definition in locals.tf

resource "aws_vpc" "vpc" {
  cidr_block           = var.vpc_cidr_block
  enable_dns_hostnames = var.enable_dns_hostnames

  tags = local.common_tags
}


resource "aws_subnet" "subnet1" {
  cidr_block              = var.vpc_subnets_cidr_blocks[0]
  vpc_id                  = aws_vpc.vpc.id
  map_public_ip_on_launch = var.map_public_ip_on_launch
  availability_zone       = data.aws_availability_zones.available.names[0]

  tags = local.common_tags
}
```

## Output Variable

- Output variables are how we get information out of Terraform
- Outputs are printed on the terminal at end of the configuration run

### Output Variable Definition

- Output variables are defined using the `output` keyword

```bash
output "name_label" {
  value = output_value // only argument required
  description = "description of the output" // optional argument
  sensitive = true | false
}
```

- Outputs can also be moved to `outputs.tf`

```bash
// outputs.tf
output "aws_instance_public_dns" {
  value = aws_instance.nginx1.public_dns
  description = "Public DNS name of the Web Server"
  sensitive = false
}
```

## Supply Value to Variable

- There are 6 different ways to pass varibales to Terraform
- The easiest way to pass a value will be to set the vaule with default argument
- Can pass the value while executing using terraform run using `-var` flag \
  `terraform run -var <variable_name=variable_value>`
- Can put all the variables and values into a file as `kay-value` pair and pass the file uinsg `-var-file` flag \
  `terraform run -var-file <variable_file>`
- Values can be passed as files also with files `.tfvars` and `.tfvars.json` but they should be in the same directory as configuration
- Values can be passed as files also with files `.auto.tfvars` and `.auto.tfvars.json` but they should be in the same directory as configuration
- Final option will be to use environment variables starting with `TF_VAR_<variable_name>`
- If variable is not defined with any of these options, Terraform will prompt for value during execution

- Variable value evaluation precedence -(from left to right)

`TF_VAR_` -> `.tfvars` or `.tfvars.json` -> `.auto.tfvars` or `auto.tfvars.json` -> `-var-file` -> `-var` -> command prompt
