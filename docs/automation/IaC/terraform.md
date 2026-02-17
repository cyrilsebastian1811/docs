# __:material-terraform: Terraform__

Terraform is an Infrastructure as Code tool that allows you to define infrastructure using declarative configuration files and manage it consistently across environments.

__Key Benefits:__

- Cloud agnostic (AWS, Azure, GCP, Kubernetes, etc.)
- Version-controlled infrastructure
- Repeatable and predictable deployments
- Dependency-aware execution

---

Core Concepts

| Concept  | Description                               |
| -------- | ----------------------------------------- |
| Provider | Plugin that interacts with cloud APIs     |
| Resource | Infrastructure component (VM, bucket, DB) |
| Module   | Reusable group of Terraform configs       |
| State    | Mapping of resources to real-world infra  |
| Plan     | Execution preview                         |
| Apply    | Executes planned changes                  |


__Typical Project Structure__

```text
terraform/
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── terraform.tfvars
└── modules/
    └── vpc/
```

!!! tip "Best Practice"

    Separate root configuration from modules for reusability and maintainability.


### Providers

Providers define which platform Terraform interacts with.

```hcl title="providers.tf"
provider "aws" {
  region = "us-east-1"
}
```

!!! note

    Providers are automatically downloaded during terraform init.

### Resources

Resources define infrastructure components.

```hcl title="modules/s3/main.tf"
resource "aws_s3_bucket" "example" {
  bucket = "my-terraform-bucket"
  acl    = "private"
}
```

__Resource Lifecycle__

``` mermaid
flowchart LR
    Write --> Init --> Plan --> Apply --> Destroy
```

### Variables

=== "Declaration Variables"

    - Declares which variables exist
    - Defines types, defaults, constraints, and descriptions
    - Is usually committed to version control

    ```hcl title="modules/ec2/variables.tf" hl_lines="18-21"
    variable "instance_type" {
        description = "EC2 instance type"
        type        = string
        default     = "t3.micro"
    }

    variable "region" {
        description = "AWS region to deploy resources"
        type        = string
        default     = "us-east-1"
    }

    variable "instance_count" {
        description = "Number of EC2 instances"
        type        = number
    }

    variable "db_password" {
        type      = string
        sensitive = true
    }
    ```

    !!! info "Key Characteristics"

        - Does not contain environment-specific values
        - Can be reused across multiple environments
        - Helps Terraform validate inputs


=== "Assignment"

    Actual values for those variables

    - Assigns actual values to declared variables
    - Values here override defaults
    - Automatically loaded by Terraform

    ```hcl title="modules/ec2/terraform.tfvars"
    instance_type  = "t2.micro"
    region         = "us-west-2"
    instance_count = 3
    ```

    !!! warning "Security Note"
        
        `terraform.tfvars` often contains sensitive data and is frequently excluded from Git.

    __Recommended Pattern__

    ```text
    envs/
    ├── dev/
    │   └── terraform.tfvars
    ├── staging/
    │   └── terraform.tfvars
    └── prod/
        └── terraform.tfvars
    ```

    ```bash
    terraform apply -var-file=envs/prod/terraform.tfvars
    ```

    !!! success

        This keeps environments isolated while reusing the same Terraform code.


=== "Using Variables"

    ```hcl title="modules/ec2/main.tf"
    resource "aws_instance" "app" {
    instance_type = var.instance_type
    }
    ```

---

!!! warning
    
    - Never commit secrets to `variables.tf` or `terraform.tfvars`.
    - Use environment variables or secret managers.

#### Precedence Order

Terraform resolves variable values in the following order (lowest → highest priority):

1. Default values in `variables.tf`
2. `terraform.tfvars`: auto loads
3. `*.auto.tfvars`: any file matching this pattern is automatically loaded by Terraform during `plan` and `apply`.
4. Environment variables (`TF_VAR_*`)
5. CLI flags (`-var`, `-var-file`). `dev.tfvars`, `prod.tfvars`, `myvars.tfvars` doesn't auto load.

!!! tip

    CLI flags always win — useful for CI/CD overrides.


### Outputs

Outputs expose useful values after deployment.

```hcl title="modules/ec2/output.tf"
output "instance_ip" {
  value = aws_instance.app.public_ip
}
```

### Modules

Modules allow reuse and composition.

__Module Structure__

```text
modules/vpc/
├── main.tf
├── variables.tf
└── outputs.tf
```

__Calling a Module__

```hcl title="**/main.tf"
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
}
```

!!! success
    
    Modules significantly reduce duplication and improve consistency.


### State Management

Terraform stores state in a state file.

__Local vs Remote State__

| Type   | Pros          | Cons              |
| ------ | ------------- | ----------------- |
| Local  | Simple        | Not collaborative |
| Remote | Team-friendly | Requires backend  |


#### Remote State Example (S3)

```hcl title="main.tf"
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
  }
}
```

!!! danger

    Never commit `terraform.tfstate` to version control.


### Terraform Workflow

``` mermaid
sequenceDiagram
    Developer->>Terraform: terraform init
    Developer->>Terraform: terraform plan
    Developer->>Terraform: terraform apply
    Terraform->>Cloud: Create / Update Resources
```

## Environments

Use workspaces or separate state files for different environments.

=== "Workspaces"

    ```bash
    terraform workspace new dev
    terraform workspace select prod
    ```

=== "Separate Directories"

    ```text
    envs/
    ├── dev/
    ├── staging/
    └── prod/
    ```

!!! info
    
    For large systems, separate directories are preferred over workspaces.


## Secrets Management

__Recommended approaches:__

- Environment variables (`TF_VAR_*`)
- Cloud secret managers
- Encrypted `.tfvars` files
- Terraform Cloud variables

```bash
export TF_VAR_db_password="supersecret"
```

## Useful Commands

| Command            | Purpose            |
| ------------------ | ------------------ |
| terraform init     | Initialize project |
| terraform plan     | Preview changes    |
| terraform apply    | Apply changes      |
| terraform destroy  | Remove infra       |
| terraform fmt      | Format files       |
| terraform validate | Validate config    |




