# Terraform

## Installion

* linux
    ```
    wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
    sudo apt update && sudo apt install terraform
    ```

* macOS
    ```
    brew tap hashicorp/tap
    brew install hashicorp/tap/terraform
    ```

* windows
    ```
    https://releases.hashicorp.com/terraform/1.10.3/terraform_1.10.3_windows_386.zip
    ```

## HCL Syntax :

* `<block>` : is a collection of key-value pairs.
* `<parameters>` : are the arguments that are passed to the block.
* `<key>` : is the name of the attribute.
* `<value>` : is the value of the attribute.
    ```
    <block> <paramaters> {
        <key> = <value>
        <key> = <value>
    }
    ```
* Example:
    - `resource` : block name
    - `local_file` (resource type) :
        * `local` : provider 
        * `file` : resource 
    - `local_sensitive_file` (resource type) : hide the content of the file in terraform apply and plan commands output
    - `res-name` : resource name
    ```hcl
    resource "local_file" "res-name" {
        filename = "/tmp/file.txt"
        content = "Hello, World!"
        file_permission = "0700"
    }
    ```
## Teraform basics

### terraform commands

* initialize the configuration
    ```sh
    terraform init
    ```
* view the configuration
    ```sh
    terraform plan
    ```
* apply the configuration
    ```sh
    terraform apply
    ```
* show the details of the cretead resources
    ```sh
    terraform show
    ```
* remove the created resources
    ```sh
    terraform destroy
    ```
* show output values
    ```sh
    terraform output
    ```
* validate the configuration files
    ```sh
    terraform validate
    ```
* beautify the configuration files
    ```sh
    terraform fmt
    ```
* refresh the state file
    ```sh
    terraform refresh
    ```
* show dependency graph
    ```sh
    terraform graph
    ```
    - use graphviz to generate a comprehensible graph
        ```sh
        terraform graph | dot -Tpng > graph.png
        ``` 
### terraform Providers

* Providers are plugins that enable terraform to interact with cloud providers, SaaS providers, and other APIs
* Each provider adds a set of resource types and data sources that Terraform can manage
* Common providers:
    - AWS
    - Azure
    - Google Cloud
    - Local
    - Docker

* Provider Configuration
    ```hcl
    # Basic provider configuration
    provider "aws" {
    region = "us-west-2"
    }

    # Multiple provider configurations
    provider "aws" {
    alias  = "us-east-1"
    region = "us-east-1"
    }

    provider "aws" {
    alias  = "us-west-2"
    region = "us-west-2"
    }
    ```

* Provider Requirements
    ```hcl
    terraform {
    required_providers {
        aws = {
        source  = "hashicorp/aws"
        version = "~> 4.0"
        }
        docker = {
        source  = "kreuzwerker/docker"
        version = "3.0.2"
        }
    }
    }
    ```
* version constraints:
    - `~> 4.0`: Any version in the 4.x range
    - `>= 3.0`: Any version greater than or equal to 3.0
    - `<= 2.0`: Any version less than or equal to 2.0
    - `!= 1.0`: Any version except 1.0
    - `~> 2.0, >= 2.3`: Any version in the 2.x range, but at least 2.3

* Provider Documentation
    - Providers are documented in the [Terraform Registry](https://registry.terraform.io/browse/providers)
    - Each provider has its own documentation with:
        - Configuration options
        - Available resources
        - Data sources
        - Examples

### terraform configuration directory structure

* Standard Terraform project structure:
    ```
    project/
    ├── main.tf         # Primary configuration
    ├── variables.tf    # Variable declarations
    ├── outputs.tf      # Output declarations
    └── providers.tf    # Provider configurations
    ```

* File purposes:
    - `main.tf`: Core resource definitions and primary infrastructure code
    - `variables.tf`: Input variable declarations with descriptions and constraints
    - `outputs.tf`: Output value definitions for resource attributes
    - `providers.tf`: Provider configurations and version constraints

* Additional common files:
    ```
    project/
    ├── terraform.tfvars    # Variable values
    ├── backend.tf          # Backend configuration
    └── versions.tf         # Terraform version constraints
    ```

* Best practices:
    - Keep related resources together in main.tf
    - Use consistent naming conventions
    - Document variables with descriptions
    - Group resources by purpose or service
    - Separate sensitive values into terraform.tfvars (git-ignored)

### terraform variables

#### variable declaration in `variables.tf`:
```hcl
variable "filename" {
    type        = string
    description = "Path to the file"
    default     = "/tmp/pets.txt"
}

variable "content" {
    type        = string
    description = "Content of the file"
    default     = "I love pets!"
}

variable "file_permission" {
    type        = string
    description = "File permissions"
    default     = "0700"
}
```
#### using variables in `main.tf`:
```hcl
resource "local_file" "pet" {
    filename        = var.filename
    content         = var.content
    file_permission = var.file_permission
}
```
- exmple :

    ```hcl
    #variables.tf
    variable "ami" {
        type        = string
        description = "AMI ID"
    }

    variable "instance_type" {
        type        = string
        description = "t2.micro"
    }
    ```
    ```hcl
    # main.tf
    resource "aws_instance" "web-server" {
        ami           = var.ami
        instance_type = var.instance_type
    }
    ```
#### Ways to set variables:

- in terraform.tfvars file:
    ```hcl
    filename = "/tmp/dogs.txt"
    content  = "I love dogs!"
    ```
- command line: 
    ```sh
    terraform apply -var="filename=/tmp/cats.txt"
    ```
- environment variables:
    ```sh
    export TF_VAR_filename="/tmp/hamsters.txt"
    ```
#### Variable types:
- string: Simple text
    ```hcl
    variable "name" {
        type    = string
        default = "John"
    }
    ```
- number: Any numeric value
    ```hcl
    variable "age" {
        type    = number
        default = 25
    }
    ```
- bool: True or false
    ```hcl
    variable "active" {
        type    = bool
        default = true
    }
    ```
- list: Ordered collection
    ```hcl
    variable "colors" {
        type    = list(string)      # string,number,bool
        default = ["red", "blue"]
    }
    ```
    * calling the variable:
        ```hcl
        # Access first element
        resource "local_file" "color_file" {
            filename = "/tmp/color.txt"
            content  = var.colors[0]  # Returns "red"
        }
        ```

- map: Key-value pairs
    ```hcl
    variable "user" {
        type = map(string)        # string,number,bool
        default = {
            name = "john"
            role = "admin"
        }
    }
    ```
    * calling the variable:
        ```hcl
        # Access map values
        resource "local_file" "user_file" {
            filename = "/tmp/user.txt"
            content  = var.user["name"]  # Returns "john"
        }
        ```
- set: unique collection (a list with no duplicate values)
    ```hcl
    variable "tags" {
        type    = set(string)      # string,number,bool
        default = ["web", "prod"]
    }
    ```
    * calling the variable:
        ```hcl
        # Access set values
        resource "local_file" "tag_file" {
            filename = "/tmp/tag.txt"
            content  = join(",", var.tags)  # Returns "web,prod"
        }
        ```
- object: complex data structure
    ```hcl
    variable "user" {
        type = object({
            name = string
            role = string
            age  = number
        })
        default = {
            name = "john"
            role = "admin"
        }
    }
    ```
    * calling the variable:
        ```hcl
        # Access object values
        resource "local_file" "user_file" {
            filename = "/tmp/user.txt"
            content  = var.user.name  # Returns "john"
        }
        ```
- tuple: ordered collection with fixed number of elements
    ```hcl
    variable "user" {
        type = tuple([string,number])
        default = ["john",25]
    }
    ```
    * calling the variable:
        ```hcl
        # Access tuple values
        resource "local_file" "user_file" {
            filename = "/tmp/user.txt"
            content  = var.user[0]  # Returns "john"
        }
        ```
#### resource attributes references :

* basic syntax: `<resource_type>.<resource_name>.<attribute>`
    - example with aws_instance: `aws_instance.web.id`
    - example with local_file: `local_file.time.content`
    - example with docker_container: `docker_container.nginx.ip_address`

* finding available attributes:
    - visit Terraform Registry provider documentation:
         * AWS: https://registry.terraform.io/providers/hashicorp/aws/latest/docs
         * Local: https://registry.terraform.io/providers/hashicorp/local/latest/docs
         * Docker: https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs

    - using terraform commands:
         ```sh
         # show all attributes of a resource
         terraform state show aws_instance.web

         # show complete state with all resources
         terraform show
         ```

* common attribute patterns across providers:
    - most resources have an `id` attribute
    - AWS resources typically have an `arn` attribute
    - resources often have a `tags` attribute for metadata
    - status-type attributes like `status`, `state`, or `condition`

* using the output of a resource as the input of another resource
* resource dependencies:
    - implicit dependencies: Terraform automatically determines dependencies when one resource references another
        ```hcl
            resource "time_static" "time_update" {
            }
            resource "local_file" "time" {
                filename = "/root/time.txt"
                content = "Time stamp of this file is ${time_static.time_update.id}" # using the output of time_static as the input of local_file
            }
        ```
    - explicit dependencies: Use `depends_on` when dependency isn't obvious from resource attributes
        ```hcl
        # EC2 instance
        resource "aws_instance" "web" {
            ami           = "ami-123456"
            instance_type = "t2.micro"
        }

        # Security group with explicit dependency
        resource "aws_security_group" "sg" {
            name = "web-sg"
            depends_on = [aws_instance.web]  # explicit dependency
        }
        ```
#### output values :

* output values are used to extract information from the terraform state these values can be used in a shell script or an ansible playbook
    ```hcl
    output <varaiable_name> {
        value = <resource_type>.<resource_name>.<attribute>
        <arguments>
    }
    ```
    - example:
        ```hcl
        output "instance_id" {
            value = aws_instance.web.id
        }
        ```

## Terraform state :

* terraform state is a JSON file that maps resource attributes to resource instances
* State file is stored locally by default in `terraform.tfstate`
* State file is used to:
    - Track resource metadata
    - Manage dependencies
    - Store resource attributes
    - Update resources
    - Destroy resources
    - Store output values
* terraform state files may contain sensitive information its not recommended to store them in a version control system like git a better approach is to store them in a remote backend like S3 or terraform cloud
* terraform state file must not be modified manually it should be managed by state management commands

## Mutible vs Immutable infrastructure :

* mutable infrastructure:
    - servers are updated in place
    - changes are made to existing servers
    - configuration drift can occur
    - difficult to rollback changes
    - difficult to scale horizontally
    - difficult to test changes
    - difficult to maintain consistency
    - difficult to recover from failures
    - difficult to automate

* immutable infrastructure:
    - servers are replaced with new instances
    - changes are made by creating new servers
    - no configuration drift
    - easy to rollback changes
    - easy to scale horizontally
    - easy to test changes
    - easy to maintain consistency
    - easy to recover from failures
    - easy to automate

* terraform is based on the immutable infrastructure model it creates new resources when changes are made to the configuration and destroys the old ones

## Meta-arguments :

### lifecycle rules :

* lifecycle rules are used to control the behavior of resources during the terraform apply and destroy commands
    - create_before_destroy: create a new resource before destroying the old one
        ```hcl
        resource "aws_instance" "web" {
            ami           = "ami-123456"
            instance_type = "t2.micro"
            lifecycle {
                create_before_destroy = true
            }
        }
        ```
    - prevent_destroy: prevent a resource from being destroyed
        ```hcl
        resource "aws_instance" "web" {
            ami           = "ami-123456"
            instance_type = "t2.micro"
            lifecycle {
                prevent_destroy = true
            }
        }
        ```
    - ignore_changes: ignore changes to specific attributes
        ```hcl
        resource "aws_instance" "web" {
            ami           = "ami-123456"
            instance_type = "t2.micro"
            tags = {
                Name = "web-server"
            }
            lifecycle {
                ignore_changes = [
                    tags
                ]
            }
        }
        ```
    
### count :

* count is used to create multiple instances of a resource
    ```hcl
    resource "local_file" "pet" {
        filename = var.filename[count.index] # access the elements of the filename variable using the count.index variable
        count= length(var.filename)  # create multiple instances of the resource based on the length of the filename variable
    }
    ```
    ```hcl
        variable "filename" {
            type = list(string)
            default = [
                "/tmp/dogs.txt",
                "/tmp/cats.txt",
                "/tmp/hamsters.txt"
            ]
        }
    ```
### for_each :
* for_each is used to create multiple instances of a resource based on a map or set only
* it is possible to use for_each with a list by converting it to a set using the toset function
    ```hcl
        resource "local_file" "pet" {
            filename = each.value
            for_each = var.filename 
            # for_each = toset(var.filename)  # convert the list to a set
        }
        ```
        ```hcl
            type = set(string)
            variable "filename" {
                default = [
                    "/tmp/dogs.txt",
                    "/tmp/cats.txt",
                    "/tmp/hamsters.txt"
                ]
            }
        ```

## data sources :

* data sources are used to fetch information from external sources 
* data sources are read-only and can be used to fetch information
* retrieves the most recent Amazon Machine Image (AMI) owned by the current AWS account, with names starting with "web-".
    ```hcl
    data "aws_ami" "web" {
        most_recent = true
        owners = ["self"]
        filter {
            name = "name"
            values = ["web-*"]
        }
    }
    ```
    - using the data source:
        ```hcl
        resource "aws_instance" "web" {
            ami           = data.aws_ami.web.id
            instance_type = "t2.micro"
        }
        ```
* data sources fetch information from external sources and are read-only.
* retrieve outputs using the syntax: `<data_source_type>.<data_source_name>.<attribute>`.

* [AWS Data Sources](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources)
* [Local Data Sources](https://registry.terraform.io/providers/hashicorp/local/latest/docs/data-sources)
* [Docker Data Sources](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs/data-sources)

* Use Terraform commands to explore data sources:
    ```sh
    terraform state show data.<data_source_type>.<data_source_name>
        terraform state show data.aws_ami.web
    ```
        ```sh
    terraform show
    ```

## Terraform with AWS :

