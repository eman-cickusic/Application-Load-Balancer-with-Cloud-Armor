# Lab 2: Terraform Network Automation

## Overview

This lab demonstrates Infrastructure as Code (IaC) using Terraform to automate the deployment of Google Cloud networking resources. You'll create custom and auto-mode networks, implement reusable modules, and deploy multi-region infrastructure.

## Architecture

The Terraform configuration will create:

- **managementnet**: Custom-mode network with subnet in Region 1
- **privatenet**: Custom-mode network with subnets in Region 1 and Region 2  
- **mynetwork**: Auto-mode network with global subnets
- **VM instances**: One in each network for connectivity testing
- **Firewall rules**: HTTP, SSH, RDP, and ICMP traffic rules

## Prerequisites

- Google Cloud Project with Compute Engine API enabled
- Cloud Shell or local environment with Terraform installed
- Basic understanding of Terraform concepts

## Project Structure

```
terraform/
├── provider.tf              # Google Cloud provider configuration
├── managementnet.tf         # Management network resources
├── privatenet.tf           # Private network resources  
├── mynetwork.tf            # Auto-mode network resources
└── instance/               # Reusable VM instance module
    └── main.tf             # VM instance module definition
```

## Task 1: Set up Terraform and Cloud Shell

### Initialize Terraform Environment

1. **Create project directory**:
```bash
mkdir tfnet
cd tfnet
```

2. **Create provider configuration**:
Create `provider.tf`:
```hcl
provider "google" {
  project = var.project_id
  region  = var.region
}

variable "project_id" {
  description = "The GCP project ID"
  type        = string
}

variable "region" {
  description = "The GCP region"
  type        = string
  default     = "us-central1"
}
```

3. **Initialize Terraform**:
```bash
terraform init
```

## Task 2: Create managementnet and its resources

### Network Configuration

Create `managementnet.tf`:

```hcl
# Create the managementnet network
resource "google_compute_network" "managementnet" {
  name                    = "managementnet"
  auto_create_subnetworks = false
}

# Create managementsubnet-us subnetwork
resource "google_compute_subnetwork" "managementsubnet-us" {
  name          = "managementsubnet-us"
  region        = "us-central1"
  network       = google_compute_network.managementnet.self_link
  ip_cidr_range = "10.130.0.0/20"
}
```

### Firewall Rules

Add to `managementnet.tf`:

```hcl
# Create firewall rule for managementnet
resource "google_compute_firewall" "managementnet-allow-http-ssh-rdp-icmp" {
  name    = "managementnet-allow-http-ssh-rdp-icmp"
  network = google_compute_network.managementnet.self_link
  
  source_ranges = ["0.0.0.0/0"]

  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }

  allow {
    protocol = "icmp"
  }
}
```

### VM Instance Module

Create `instance/main.tf`:

```hcl
variable "instance_name" {
  description = "Name of the VM instance"
  type        = string
}

variable "instance_zone" {
  description = "Zone for the VM instance"
  type        = string
}

variable "instance_type" {
  description = "Machine type for the VM instance"
  type        = string
  default     = "e2-standard-2"
}

variable "instance_subnetwork" {
  description = "Subnetwork for the VM instance"
  type        = string
}

resource "google_compute_instance" "vm_instance" {
  name         = var.instance_name
  zone         = var.instance_zone
  machine_type = var.instance_type

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    subnetwork = var.instance_subnetwork
    
    access_config {
      # Allocate ephemeral external IP
    }
  }

  tags = ["http-server"]
}
```

### VM Instance Configuration

Add to `managementnet.tf`:

```hcl
# Create managementnet VM instance
module "managementnet-us-vm" {
  source              = "./instance"
  instance_name       = "managementnet-us-vm"
  instance_zone       = "us-central1-c"
  instance_subnetwork = google_compute_subnetwork.managementsubnet-us.self_link
}
```

### Deploy managementnet

```bash
terraform fmt
terraform init
terraform plan
terraform apply
```

## Task 3: Create privatenet and its resources

Create `privatenet.tf`:

```hcl
# Create privatenet network
resource "google_compute_network" "privatenet" {
  name                    = "privatenet"
  auto_create_subnetworks = false
}

# Create privatesubnet-us subnetwork
resource "google_compute_subnetwork" "privatesubnet-us" {
  name          = "privatesubnet-us"
  region        = "us-central1"
  network       = google_compute_network.privatenet.self_link
  ip_cidr_range = "172.16.0.0/24"
}

# Create privatesubnet-eu subnetwork
resource "google_compute_subnetwork" "privatesubnet-eu" {
  name          = "privatesubnet-eu"
  region        = "europe-west1"
  network       = google_compute_network.privatenet.self_link
  ip_cidr_range = "172.20.0.0/24"
}

# Create firewall rule for privatenet
resource "google_compute_firewall" "privatenet-allow-http-ssh-rdp-icmp" {
  name    = "privatenet-allow-http-ssh-rdp-icmp"
  network = google_compute_network.privatenet.self_link
  
  source_ranges = ["0.0.0.0/0"]

  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }

  allow {
    protocol = "icmp"
  }
}

# Create privatenet VM instance
module "privatenet-us-vm" {
  source              = "./instance"
  instance_name       = "privatenet-us-vm"
  instance_zone       = "us-central1-c"
  instance_subnetwork = google_compute_subnetwork.privatesubnet-us.self_link
}
```

### Deploy privatenet

```bash
terraform fmt
terraform plan
terraform apply
```

## Task 4: Create mynetwork and its resources

Create `mynetwork.tf`:

```hcl
# Create auto-mode network
resource "google_compute_network" "mynetwork" {
  name                    = "mynetwork"
  auto_create_subnetworks = true
}

# Create firewall rule for mynetwork
resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
  name    = "mynetwork-allow-http-ssh-rdp-icmp"
  network = google_compute_network.mynetwork.self_link
  
  source_ranges = ["0.0.0.0/0"]

  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }

  allow {
    protocol = "icmp"
  }
}

# Create mynetwork VM instances
module "mynet-us-vm" {
  source              = "./instance"
  instance_name       = "mynet-us-vm"
  instance_zone       = "us-central1-c"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}

module "mynet-eu-vm" {
  source              = "./instance"
  instance_name       = "mynet-eu-vm"
  instance_zone       = "europe-west1-b"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}
```

### Deploy mynetwork

```bash
terraform fmt
terraform plan
terraform apply
```

## Verification and Testing

### Check Resources in Console

1. **VPC Networks**: Navigate to VPC Network > VPC networks
   - Verify `managementnet`, `privatenet`, and `mynetwork`
   - Check subnet configurations

2. **Firewall Rules**: Navigate to VPC Network > Firewall
   - Verify firewall rules for each network

3. **VM Instances**: Navigate to Compute Engine > VM instances
   - Verify all VM instances are running
   - Note internal IP addresses

### Test Connectivity

1. **SSH to managementnet-us-vm**:
```bash
gcloud compute ssh managementnet-us-vm --zone=us-central1-c
```

2. **Test connectivity to privatenet (should fail)**:
```bash
ping -c 3 [privatenet-us-vm-internal-ip]
```

3. **SSH to mynet-us-vm**:
```bash
gcloud compute ssh mynet-us-vm --zone=us-central1-c
```

4. **Test connectivity to mynet-eu-vm (should succeed)**:
```bash
ping -c 3 [mynet-eu-vm-internal-ip]
```

## Key Terraform Concepts

### Resource Dependencies

Terraform automatically handles resource dependencies using:

- **Implicit Dependencies**: References like `google_compute_network.managementnet.self_link`
- **Explicit Dependencies**: Using `depends_on` attribute when needed

### Modules

Benefits of using modules:
- **Reusability**: Same module for multiple VM instances
- **Maintainability**: Centralized configuration
- **Abstraction**: Hide complexity behind simple interfaces

### State Management

Terraform tracks resource state:
- **terraform.tfstate**: Stores current state
- **State Locking**: Prevents concurrent modifications
- **Remote State**: Recommended for team environments

## Terraform Commands Reference

### Basic Commands

```bash
# Initialize Terraform
terraform init

# Format configuration files
terraform fmt

# Validate configuration
terraform validate

# Create execution plan
terraform plan

# Apply changes
terraform apply

# Destroy resources
terraform destroy

# Show current state
terraform show

# List resources in state
terraform state list
```

### Advanced Commands

```bash
# Target specific resources
terraform apply -target=google_compute_network.managementnet

# Import existing resources
terraform import google_compute_network.example projects/PROJECT_ID/global/networks/NETWORK_NAME

# Refresh state from actual infrastructure
terraform refresh

# View specific resource details
terraform state show google_compute_network.managementnet
```

## Best Practices

### Configuration Organization

1. **Separate Files**: Group related resources
2. **Consistent Naming**: Use clear, descriptive names
3. **Comments**: Document complex configurations
4. **Variables**: Use variables for reusable values

### Security Considerations

1. **Least Privilege**: Minimize firewall rule scope
2. **Network Isolation**: Use separate networks for different environments
3. **State Security**: Protect Terraform state files
4. **Credential Management**: Use service accounts, not user credentials

### Version Control

```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
}
```

## Troubleshooting

### Common Issues

1. **Provider Authentication**:
```bash
gcloud auth application-default login
```

2. **API Not Enabled**:
```bash
gcloud services enable compute.googleapis.com
```

3. **Resource Already Exists**:
   - Use `terraform import` to import existing resources
   - Or rename resources in configuration

4. **State Lock Issues**:
```bash
terraform force-unlock LOCK_ID
```

### Debugging

1. **Enable Debug Logging**:
```bash
export TF_LOG=DEBUG
terraform apply
```

2. **Validate Configuration**:
```bash
terraform validate
terraform plan -detailed-exitcode
```

## Cleanup

To destroy all resources:

```bash
terraform destroy
```

**Warning**: This will delete all resources managed by Terraform. Confirm carefully!

## Advanced Topics

### Remote State Backend

Configure remote state storage:

```hcl
terraform {
  backend "gcs" {
    bucket = "tf-state-bucket"
    prefix = "terraform/state"
  }
}
```

### Workspaces

Manage multiple environments:

```bash
terraform workspace new production
terraform workspace select production
terraform apply -var-file="production.tfvars"
```

### Data Sources

Reference existing resources:

```hcl
data "google_compute_network" "existing" {
  name = "existing-network"
}

resource "google_compute_subnetwork" "subnet" {
  network = data.google_compute_network.existing.self_link
  # ... other configuration
}
```

## Learning Outcomes

After completing this lab:

- **Infrastructure as Code**: Understand IaC principles and benefits
- **Terraform Workflow**: Master init, plan, apply, destroy cycle
- **Resource Management**: Learn dependency handling and state management
- **Module Development**: Create reusable, maintainable modules
- **Network Architecture**: Understand GCP networking concepts
- **Best Practices**: Apply security and organization best practices

---

This completes Lab 2 of the Terraform Network Automation implementation.