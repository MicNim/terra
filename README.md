## Contents

  - [What is Teraform](##What-is-Terraform)
  - [Infrastructure](##Infrastructure)
  - [IAC Infrastructure as Code](##IAC-Infrastructure-as-Code)
  - [Install Terraform](##Install-Terraform)
  - [Create First Resource](##Create-First-Resource)
    - [provider.tf](###provider.tf)
    - [main.tf](###main.tf)
  - [Init Plan Deploy Destroy](##Init-Plan-Deploy-Destroy)
  - [State File](##State-File)
  - [How to Update Deployed Resources](##How-to-Update-Deployed-Resources)
  - [Variables](##Variables)
  - [Resource Attributes](##Resource-Attributes)
  - [Outputs](##Outputs)
  - [Loops](##Loops)
  - [Modules](##Modules)
  - [Configuring One Module With Another Modules Output](##Configuring-One-Module-With-Another-Modules-Output)
  - [Environment](##Environment)
  - [Files to Ignore](##Files-to-Ignore)
  - [Interesting Resources](##Interesting-Resources)

## What is Terraform 

Used for infrastructure as code developed by hashicorp

## Infrastructure

  * virtual servers
  * azure & gcp resources 
  * lots of support providers

  https://www.terraform.io/docs/providers/index.html

## IAC Infrastructure as Code

  Instead of logging in to the gui or the portal, written in code, similar to
  json.

  ```
  resource "google_storage_bucket" "state_bucket" {
    name     = "terraform-state-bucket"
    location = var.region
  }
  ```

## Install Terraform

  https://learn.hashicorp.com/terraform/getting-started/install.html

  [tfswitch](https://warrensbox.github.io/terraform-switcher/)

  [aliases](https://github.com/zer0beat/terraform-aliases)

  [awesome](https://github.com/shuaibiyy/awesome-terraform)

  [awesome](https://github.com/Azure/awesome-terraform)

## Create First Resource

  First create a folder called terraform.

  Next create two files:

  * main.tf
  
  * provider.tf

  ### provider.tf

  The provider informs the terraform client which service to interact with,
  this also installs the required plugins for that service.

  ```
  provider "aws" {
    access_key = ""
    secret_key = ""
    region = ""
  }
  ```

  https://www.terraform.io/docs/providers/aws/index.html

  https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html

  ### main.tf

  The main file is where resources should be defined, in larger projects
  it may make sense to separate resources into logical groups.

  ```
  resource "aws_vpc" "my_vpc" {
    cidr_block       = "10.0.0.0/16"
    instance_tenancy = "default"

    tags = {
      Name = "my_vpc"
    }
  }
  ```
  
  https://www.terraform.io/docs/providers/aws/r/vpc.html

## Init, Plan, Deploy & Destroy 

  Open terminal and navigate to terraform directory. 

  Run ```terraform init``` this will pull down any modules or provider plugins.

  Run ```terraform plan``` this will look at the state file compared to the 
  available specs to see if there are any changes to be made.

  Run ```terraform apply``` to execute the plan.

  Run ```terraform destroy``` to remove the deployed resources.

## State File

  Anything that is created with terraform it is tracked in the state file.
  This is the source of all truth for terraform to interact with deployed 
  resources.
  If the state file is lost then the resources will become orphaned and will 
  no longer be controllable with terraform.

  Open the statefile to see what happens when you run:
    ```terraform apply --auto-approve```

  Run ```terraform state list``` to see current resources.
 
  Run ```terraform state show``` to see an easy to read detailed version of 
  the state file.

  https://www.terraform.io/docs/state/index.html

## How to update deployed resources

  Change the tag deployed with the original vpc resource in main.tf.

  ```
    resource "aws_vpc" "my_vpc" {
    cidr_block       = "10.0.0.0/16"
    instance_tenancy = "default"

    tags = {
+++   Name = "vpc_001"
---   Name = "my_vpc"
    }
  }
  ```

  Run ```terraform plan``` to see resource modificaitons.

  Run ```terraform apply --auto-approve``` to execute the changes.

## Variables

  Create a variables.tf file, use this file to store definitions of the 
  variables used to configure resources.

  ```
    variable "vpc_name" {
      type = string
      default = "default_name"
    }
  ```

  Open main.tf and update the tag block with a reference to the variable.

  ```
  resource "aws_vpc" "my_vpc" {
    cidr_block       = "10.0.0.0/16"
    instance_tenancy = "default"

    tags = {
+++   Name = var.vpc_name
---   Name = "vpc_001"
    }
  }
  ```

  Try referencing different variable types list and map.

  ```
  variable "mylist" {
    type    = list
    default = ["value1", "value2"]
  }
  
  variable "mymap" {
    type = map

    default = {
      key1 = "value1"
      key2 = "value2" 
    }
  }
  ```

  List ref: ```var.mylist[0]```

  Map ref: ```var.mymap["key1"]```

  If you do not specify a default variable you will be prompted when applying 
  the current specs. There are other methods for configuring variables in
  practice.

  https://www.terraform.io/docs/configuration/variables.html

  Configure the name tag using a .tfvars file & environment variables.

## Resource Attributes

  Each resource has a set of attributes. These attributes can be referenced to 
  provide configuration in other resources within a module.

  Add a subnet resource to main.tf and configure the vpc_id to the id attribute
  of the vpc.

  ```
  resource "aws_subnet" "my_subnet" {
    vpc_id     = aws_vpc.my_vpc.id
    cidr_block = "10.0.1.0/24"
  }
  ```

  https://www.terraform.io/docs/providers/aws/r/vpc.html#attributes-reference

## Outputs

  Output values are module or resource return values, they can be used to:

  * expose these values in the CLI after ```apply``` or ```outputs``` is run

  * a child module can use outputs to expose a subset of its resource attributes to a parent module.

  * when using remote state, outputs can be accessed by other configurations via a terraform_remote_state data source.

  Create output.tf and configure it to output the id of the vpc.

  ```
  output "vpc_id" {
    value = aws_vpc.my_vpc.id
  }
  ```

  Run ```terraform apply --auto-approve``` to add the output to the state.

  The vpc id will be printed in standard out when the command is completed.

  https://www.terraform.io/docs/configuration/outputs.html

## Loops

  There are a few different looping options available each intended for a
  different scenario:

  * count parameter: loop over resources.

  * for_each expressions: loop over resources and inline blocks 
  within a resource.

  * for expressions: loop over lists and maps.

  ```
  resource "aws_instance" "my_vm" {
    count = 3
    ami           = "ami-0a8cd349f3bde8bc8"
    instance_type = "t2.micro"

    subnet_id = aws_subnet.subnet.id

    tags = {
      Name = "my_vm"
    }
  }
  ```

## Modules

  Modules are containers for groups of resources.

  * Input variables to accept values from the calling module.
 
  * Output values to return results to the calling module, 
  which it can then use to populate arguments elsewhere.
  
  * Resources to define one or more infrastructure objects that 
  the module will manage.

  A good module should raise the level of abstraction by describing a 
  new concept in your architecture that is constructed from resource types 
  offered by providers.

  Calling child modules:

  ```
  module "vpc" {
  source = "./modules/vpc"

    vpc_name = "my_vpc_module"
  }
  ```

  Run ```terraform init``` to prepare the modules.

  Run ```terraform apply``` to deploy the modules.

## Configuring One Module With Another Modules Output

  Sometimes you will want to configure a modules resources with the attributes 
  of resources from other modules. It's not possible to reliably predict the 
  contents of these attributes and much less desirable to hard code these into
  your configuration.
  
  First the module you wish to output values from should be configured with an 
  output in outputs.tf:

  ```
  output "vpc_id" {
    value = aws_vpc.my_vpc.id
  }
  ```

  Then configure the module which is going to recieve this configuration with 
  an input variable in variables.tf.

  ```
  variable "vpc_id" {
    type = string
  }
  ```

  Make sure that the resource you intend to configure has the variable
  as the correct arguments value.

  ```
  resource "aws_subnet" "my_subnet" {
    vpc_id     = var.vpc_id
    cidr_block = "10.0.1.0/24"
  }
  ```

  Then specify the output as an input variable in the parent module.

  ```
  module "subnet" {
    source = "./subnet"

    vpc_id = module.vpc.vpc_id
  }
  ```

  https://www.terraform.io/docs/modules/index.html

## Environments

  When it comes to deploying resources to different environments it sometimes
  makes sense to configure environments as parent modules, 
  they will then import & configure any child module specified.

  ```
    ├── README.md
    ├── environments/
    │   ├── shared/
    │   │   ├── README.md
    │   │   ├── variables.tf
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    │   ├── development/
    │   │   ├── README.md
    │   │   ├── variables.tf
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    ├── modules/
    │   ├── vpc/
    │   │   ├── README.md
    │   │   ├── variables.tf
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    │   ├── ec2/
    │   │   ├── README.md
    │   │   ├── variables.tf
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    │   ├── s3/
    │   │   ├── README.md
    │   │   ├── variables.tf
    │   │   ├── main.tf
    │   │   ├── outputs.tf
  ```

## Files to Ignore

  A basic ignore file for a terraform project:

  ```
  # Local .terraform directories
  **/.terraform/*

  # .tfstate files
  *.tfstate
  *.tfstate.*

  # Exclude all .tfvars files, which are likely to contain sentitive data, such as
  # password, private keys, and other secrets. These should not be part of version 
  # control as they are data points which are potentially sensitive and subject 
  # to change depending on the environment.
  #
  *.tfvars
  ```

## Interesting Resources

https://blog.gruntwork.io/a-comprehensive-guide-to-terraform-b3d32832

https://www.hashicorp.com/resources/evolving-infrastructure-terraform-opencredo/