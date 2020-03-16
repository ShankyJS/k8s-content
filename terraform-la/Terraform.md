# What is Terraform?

Terraform is a tool for building infrastructure.

- Allows simple version control.
- Available as a open-source or enterprise software.
- Supports many popular Cloud Providers as:
  * AWS
  * Azure
  * GCP 
  * DO

Terraform is Infrastructure as Code (IaC).
  - Idempotent: You can apply your results multiple times but if your configuration is there it wouldn't do any update.
  - High-level syntax: Hashicorp high level syntax.
  - Easily Reusable: Don't repeat your self, the clear syntax of Terraform let's you use reuse the same code for various projects.

Execution plans: 
  - Show the intent of the deploy
  - Can help ensure everything in the development is intentional.

Resource graph: 
  - Illustrates all changes and dependencies.

## Use cases

Some use cases for Terraform: 
 - Hybrid clouds:
   * Cloud-agnostic.
   * Allows deployments to multiple providers simultaneously. 

 - Multi-tier architecture:
   * Allows deployment of several layers of architecture.
   * Is usually able to automatically deploy in the correct order.
   
 - Software-defined networking.
   * Able to deploy network architecture as well.

### Terraform Vs. Other Software.

Terraform is a high-level infrastructure orchestration tool.

 - Puppet, chef, and ansible are configuration management tools.
    * Not intended for configuration management. 
i   * Provides "provisioners" that can all these tools to perform the CM duties.
 
 - CloudFormation and other IaC tools:
    * Many other tools are vendor-locked and only support one vendor.
    * There are some tools that are similar to Terraform.

 - Boto and other lower-leel tools.
    * Terraform is a higher-level tool than tools such as Boto, which makes defining infrastructure easier. 

## Terraform Commands.

Common commands:
apply: Builds or changes infrastructure
console: Interactive console for Terraform interpolations
destroy: Destroys Terraform-managed infrastructure
fmt: Rewrites configuration files to canonical format
get: Downloads and installs modules for the configuration
graph: Creates a visual graph of Terraform resources
import: Imports existing infrastructure into Terraform
init: Initializes a new or existing Terraform configuration
output: Reads an output from a state file
plan: Generates and shows an execution plan
providers: Prints a tree of the providers used in the configuration
push: Uploads this Terraform module to Terraform Enterprise to run
refresh: Updates local state file against real resources
show: Inspects Terraform state or plan
taint: Manually marks a resource for recreation
untaint: Manually unmarks a resource as tainted
validate: Validates the Terraform files
version: Prints the Terraform version
workspace: Workspace management

***Set up the environment:***


Create a Terraform script:

```
vi main.tf
```

main.tf contents:

````
# Download the latest Ghost image
resource "docker_image" "image_id" {
  name = "ghost:latest"
}
````

Initialize Terraform:

````
terraform init
````

List providers in the folder:

````
ls .terraform/plugins/linux_amd64/
````

List providers used in the configuration:

terraform providers
Terraform Plan:

terraform plan
Useful flags for plan:

````
-out=path: Writes a plan file to the given path. This can be used as input to the "apply" command.
-var 'foo=bar': Set a variable in the Terraform configuration. This flag can be set multiple times.
````

terraform apply
Useful flags for apply:

````
-auto-approve: This skips interactive approval of plan before applying.
-var 'foo=bar': This sets a variable in the Terraform configuration. It can be set multiple times.
````

List the Docker images:

````
docker image ls
````

````
terraform plan -out=tfplan
````

Applying a plan:

````
terraform apply tfplan
````

Show the Docker Image resource:

````
terraform show
````

## HashiCorp configuration Language. 

The syntax of Terraform configurations is called HashiCorp Configuration Language (HCL). It is meant to strike a balance between being human-readable and editable, and being machine-friendly. For machine-friendliness, Terraform can also read JSON configurations. For general Terraform configurations, however, we recommend using the HCL Terraform syntax.

### Terraform code files
The Terraform language uses configuration files that are named with the .tf file extension. There is also a JSON-based variant of the language that is named with the .tf.json file extension.

### Terraform Syntax

Here is an example of Terraform's HCL syntax:

````
resource "aws_instance" "example" {
  ami = "abc123"

  network_interface {
    # ...
  }
}

````

### Syntax reference:
Single line comments start with #.
Multi-line comments are wrapped with /* and */.
Values are assigned with the syntax of key = value.
Strings are in double-quotes.
Strings can interpolate other values using syntax wrapped in ${}, for example ${var.foo}.
Numbers are assumed to be base 10.
Boolean values: true, false
Lists of primitive types can be made with square brackets ([]), for example ["foo", "bar", "baz"].
Maps can be made with braces ({}) and colons (:), for example { "foo": "bar", "bar": "baz" }.

### Style Conventions:
Indent two spaces for each nesting level.
With multiple arguments, align their equals signs.


#### Setup the environment:

````
cd terraform/basics
````

### Deploying a container using Terraform
Redeploy the Ghost image:

````
terraform apply
````

main.tf new  contents:

````
# Download the latest Ghost image
resource "docker_image" "image_id" {
  name = "ghost:latest"
}


# Start the Container
resource "docker_container" "container_id" {
  name  = "ghost_blog"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "2368"
    external = "80"
  }
}
````

List the Docker containers:

````tf
docker container ls
````

Access the Ghost blog by opening a browser and go to:

```http:://[SWAM_MANAGER_IP]```_

Cleaning up the environment
Reset the environment:

````tf
terraform destroy --auto-approve
````

## Working with the Terraform console

Enter:

````tf
terraform console
````

Returning the value of the ip address:

````tf
docker_container.container_id.ip_addresss
````

The correct syntax for the terraform interpolations is: ```object_type.object_name.attribute```

## Output the name and the IP of the GHost blog container

````tf
# Download the latest Ghost Image
resource "docker_image" "image_id" {
  name = "ghost:latest"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "blog"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "2368"
    external = "80"
  }
}

#Output the IP Address of the Container
output "ip_address" {
  value       = "${docker_container.container_id.ip_address}"
  description = "The IP for the container."
}

#Output the Name of the Container
output "container_name" {
  value       = "${docker_container.container_id.name}"
  description = "The name of the container."
}
````

## Using input variables as references

Input variables serve as parameters for a TF file. A variable block configures a single input for a Terraform Module. Each block declares a single variable.

**Syntax:**

````tf
variable [name] {
  [OPTION]  = "[VALUE]"
}
````

**Arguments:**

Within the block body (between the curly braces {} is a configuration for the variable, which accepts some arguments.)

- type: (Optional): If set, this defines the type of the variable. Valid values are string, list and map.
- Default (optional): This sets a default value for the variable. If no default is provided, Terraform will raise an error if a value is not provided by the caller.
- Description (optional): A human-friendly description for the variable.

If we don't define the "default" argument for a variable, it will be prompted when we apply the Terraform plan, to skip this we can pass the variable by an argument of the apply command.

````tf
terraform apply --var "ext_port:8080"
````

Content with variables:

````TF
#Define variables
variable "image_name" {
  description = "Image for container."
  default     = "ghost:latest"
}

variable "container_name" {
  description = "Name of the container."
  default     = "blog"
}

variable "int_port" {
  description = "Internal port for container."
  default     = "2368"
}

variable "ext_port" {
  description = "External port for container."
  default     = "80"
}

# Download the latest Ghost Image
resource "docker_image" "image_id" {
  name = "${var.image_name}"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "${var.container_name}"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "${var.int_port}"
    external = "${var.ext_port}"
  }
}

#Output the IP Address of the Container
output "ip_address" {
  value       = "${docker_container.container_id.ip_address}"
  description = "The IP for the container."
}

output "container_name" {
  value       = "${docker_container.container_id.name}"
  description = "The name of the container."
}
````

We can override the values of a variable too.

````TF
terraform apply -var 'container_name=ghost_blog' -var 'ext_port=8080'
````

If we created a variable without default value we will need to delete is passing the value as a parameter of the destroy command.

````TF
terraform destroy --var "ext_port=8080"
````
