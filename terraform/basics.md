# docs: terraform/basics
#terraform #basics

## HCL (Hashicorp Configuration Language)

- **Block-Name**: `resource`
- **Provider**: local
- **Resource Type**: file
- **Resource Name**: testfile
- **Arguments**: `filename`, `content`

```
resource "local_file" "testfile" {
	filename = "/home/testfile.txt"
	content = "Hello World!"
}
```

## Workflow

```bash
terraform init
terraform plan
terraform apply
```

other commands

```bash
terraform show
terraform destroy
```

## Terraform Providers

- **Registry**: default `registry.terraform.io`
- **Organizational Namespace**: e.g. hashicorp
- **Type**: name of provider (e.g. `local`)

`registry.terraform.io/hashicorp/local`

- plugins are stored after the `terraform init` command in the local directory under `.terraform/plugins`

## Configuration Directory

- `main.tf`: Main Configuration file containing resource definition
- `variables.tf`: contains variable declarations
- `outputs.tf`: contains outputs from resources
- `provider.tf`: contains provider definition

## Variables

- `default` (optional) -> interactive or as parameter
- `type` (optional)
	- `string`
	- `number`
	- `bool`
	- `any` (default)
	- `list`
	- `map`
	- `object`
	- `tuple`
- `description` (optional)

### Variable Types

- string
```
variable "filename" {
	default = "/root/testfile.txt"
	type = string
	description = "the path of the file"
}

resource "local_file" "testfile" {
	filename = var.filename
}
```

- list
```
variable "prefix" {
	default = ["Cat", "Dog", "Cow"]
	type = list # or list(string)
}

resource "random_pet" "pet" {
	prefix = var.kind[0] # Cat
}
```

- set (a list without duplicate values)
```
variable "number" {
	default = ["1", "2", "3"]
	type = set(number)
}

resource "random_pet" "pet" {
	prefix = var.number[0] # 1
}
```

- tuples (a list with elements of different  types)
```
variable kitty {
	type = tuple([string, number, bool])
	default = ["cats", 7, true]
}
```

- map
```
variable file-content {
	type = map # or map(string)
	default = {
		"statement1" = "Hello World!"
		"statement2" = "Hello John!"
	}
}

resource "local_file" "testfile" {
	filename = "/root/testfile"
	content = var.file-content["statement1"] # Hello World!
}
```

- objects
```
variable "vehicle" {
	type = object({
		name = string
		color = string
		seats = number
		addons = list(string)
		approved = bool
	})
	
	default = {
		name = "car"
		color = "red"
		seats = 4
		addons = ["air conditioner", "seat heating"]
		approved = true
	}
}
```

### Using Variables

- if the `default` argument is missing, the variable has to be set in `terraform apply` in an interactive mode
- use of commandline flags `terraform apply -var "filename=/root/testfile.txt" -var "content=Hello World"`
- use of ENV-Variables `export TF_VAR_filename=/root/testfile.txt`
- Autoloading with of Variable Definition Files 
	- `terraform.tfvars` or `terraform.tfvars.json`
	- all files ending with `*.auto.tfvars` or `*.auto.tfvars.json`
	- or with parameter `-var-file variables.tfvars`
```
filename = "/root/testfile.txt"
content = "Hello World"
```

**Variable Definition Precedence**
1. Environment Variables
2. `terraform.tfvars`
3. `*.auto.tfvars` (alphabetical order)
4. `-var` or `-var-file` (command-line flags)

## Ressource Attributes

- references to resource attributes -> implicity dependency

```
resource "local_file" "pet" {
	filename = var.filename
	content = "My favorite pet is ${random_pet.my-pet.id}"
}

resource "random_pet" "my-pet" {
	prefix = var.prefix
	separator = var.separator
	length = var.length
}
```

## Resource Dependencies

- explicit dependency:

```
resource "local_file" "pet" {
	filename = var.filename
	content = "My favorite pet is ${random_pet.my-pet.id}"
	depends_on = [
		random_pet.my-pet
	]
}

resource "random_pet" "my-pet" {
	prefix = var.prefix
	separator = var.separator
	length = var.length
}
```

## Output Variables

- output is shown in `terraform apply`  or with `terraform output` / `terraform output <output-name>`

```
output pet-name {
	value = random_pet.my-pet.id
	description = "..."
}
```

## Terraform State

- after first apply, the ressource `terraform.tfstate` is created
- single point of truth for terraform to know what is deployed in the real world
- contains sensitive data

## Terraform commands

- `terraform validat`: validate configuration files
- `terraform fmt`: format all files
- `terraform show`: display current state of ressources
- `terraform providers`: show all providers
- `terraform output`: print output definitions
- `terraform refresh`: sync terraform with real world infrastructure (update state file, is also called with `plan` and `apply`)
- `terraform graph`: show dependencies in configuration -> `terraform graph | dot -Tsvg > graph.svg`

## Mutable vs Immutable Infrastructure

- **Immutable Infrastructue**: Infrastructure is replaced with a new one -> a file is deleted and then newly created instead of updated
- this is the default approach of terraform

## Lifecycle Rules

- `lifecycle` argument, available is
	- `create_before_destroy`
	- `prevent_destroy`
	- `ignore_changes`

```
resource "local_file" "pet" {
	filename = "/root/pets.txt"
	lifecycle {
		prevent_destroy = true
	}
}

resource "local_file" "pet" {
	filename = "/root/pets.txt"
	lifecycle {
		ignore_changes = [
			tags,id
		]
		# or for all arguments
		ignore_changes = all
	}
}
```

## Data Sources

- read attributes from objects created outside of terraform
- only "reads" Infrastructure

```
data "local_file" "dog" {
	filename = "/root/dog.txt"
}

resource "local_file" "pet" {
	filename = "/root/pets.txt"
	content = data.local_file.dog.content
}
```

## Meta Arguments

- change behaviour of a resource
- available types:
	- `lifecycle`
	- `depends_on`
	- `count`
	- `for_each`

- count
	- resources are created as a list in the state file -> for-each is better because of the index of the list
```
variable "filename" {
	type=list(string)
	default = [
		"/root/pets.txt" # index 0
		"/root/dogs.txt" # index 1
		"/root/cats.txt" # index 2
	]
}

resource "local_file" "pet" {
	filename = var.filename[count.index]
	count = length(var.filename)
}
```

- for each (works only with a map or set)
	- resources are created as a map in the state file
```
variable "filename" {
	type=list(string)
	default = [
		"/root/pets.txt"
		"/root/dogs.txt"
		"/root/cats.txt"
	]
}

resource "local_file" "pet" {
	filename = each.value
	for_each = toset(var.filename)
}
```

## Version Constraints

- use fix version of provider

```
terraform {
	required_providers {
		local = {
			source = "hashicorp/local"
			version = "1.4.0" # or "!= 2.0.0" / "< 1.4.0" etc.
		}
	}
}
```

## Remote State

- use a remote backend for the state-file

```
terraform {
	backend "s3" {
		bucket = "bucket01"
		key = "states/terraform.tfstate"
		region = "us-wes-1"
		dynamodb_tyble = "state-locking"
	}
}
```

## State Commands

- `terraform state list`: get all resources (without details)
- `terraform state show`: show attributes of ressource
- `terraform state mv`: move a resource in the state file (e.g. rename)
- `terraform state pull`: download & display remote state file
- `terraform state rm`: remove a ressource

## Terraform Provisioners

- use rarely
- no provisioner information in plan
- Network Connectivity and Authentication must be set up

```
resource "..." "server" {
	provisioner "remote-exec" {
		inline = [ "sudo apt upgrade" ]	
	}
	provisioner "local-exec" {
		when = destroy
		command = "echo server destroyed"
		on_failure = fail # default behaviour
	}
	connection {
		type = ssh
		host = 10.10.10.10
		user = "ubuntu"
		private_key = file("/root/.ssh/key")
	}
}
```

## Taint

- if a `terraform apply` fails for a specific resource, this ressource is "tainted"
- the next time, `terraform plan` and `terraform apply` is called, this resource will be recreated
- a ressource can be manually tainted to "force-recreate" it or also untainted

```bash
terraform taint <ressource-name>
terraform untaint <ressource-name>
```

## Debugging

- set ENV `TF_LOG` to:
	- INFO, WARNING; ERROR, DEBUG or TRACE

## Import

- import a ressource, not created with terraform

```
terraform import aws_instance.webserver-2 i-0263f...
```

## Modules

- benefit: simpler configuration files, lower risk, re-usability, standardized

- local module
```
module "dev-webserver" {
	source = "../aws-instance
	# insert variables for the module
	app_region = "us-east-1"
}
```

- module from the registry
```
module "security-group" {
	source = "terraform-aws-modules/security-group/aws/modules/ssh"
	version = "3.16.0"
	# insert required variables here
}
```

- `terraform get`: downloads only the modules from the registry

## Functions

- `file`: read data from file
- `length`: get length of map or list
- `toset`: convert list to set

- experiment with functions
```bash
terraform console

> file("/root/main.tf")
(...)
> length(var.region)
(...)
> toset(var.region)
(...)
```

- **Numeric Functions**: `max()`, `min()`, `ceil()`, `floor()` 
- **String Functions**: `split()`, `lower()`, `upper()`, `title()`, `substr()`, `join()`
- **Collection Functions**: `length()`, `index()`, `element()`, `contains()`
- **Map Functions**: `keys()`, `values()`, `lookup()`

## Operators & Conditional Expressions

- **Nummeric Operators**: `+ - * /`
- **Equality Operators**: `== !=`
- **Comparison Operators**: `> < >= <=`
- **Logical Operators**: `&& || !`

- **Conditional Expressions**: `condition ? true_val : false_val`

## Terraform Workspaces (OSS)

- same configuratin directory for different projects
- store state file per workspace in the directory `terraform.tfstate.d`

```bash
terraform workspace new ProjectA

terraform workspace list

terraform workspace select ProjectB
```