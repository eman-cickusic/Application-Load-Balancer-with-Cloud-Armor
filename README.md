# Google Cloud Application Load Balancer with Cloud Armor

This repository contains a comprehensive guide and automation scripts for implementing Google Cloud Application Load Balancing with Cloud Armor security policies. The project demonstrates global load balancing, backend configuration, stress testing, and IP denylisting capabilities.

## Video

https://youtu.be/KZa1gWJfLLM

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Lab 1: Application Load Balancer with Cloud Armor](#lab-1-application-load-balancer-with-cloud-armor)
- [Lab 2: Terraform Network Automation](#lab-2-terraform-network-automation)
- [Getting Started](#getting-started)
- [Cleanup](#cleanup)
- [Contributing](#contributing)
- [License](#license)

## Overview

This project implements two main components:

1. **Application Load Balancer with Cloud Armor**: Configure global load balancing with security policies
2. **Terraform Network Automation**: Infrastructure as Code for network deployment

### Key Features

- Global Application Load Balancer with IPv4 and IPv6 support
- Multi-region backend instances with autoscaling
- Cloud Armor security policies for IP denylisting
- Stress testing capabilities
- Terraform automation for network infrastructure
- Comprehensive monitoring and logging

## Architecture

The solution deploys:

- **Frontend**: Global Application Load Balancer with IPv4/IPv6 endpoints
- **Backend**: Managed instance groups across multiple regions
- **Security**: Cloud Armor policies for traffic filtering
- **Monitoring**: Health checks and logging integration
- **Infrastructure**: Terraform-managed network resources

## Prerequisites

- Google Cloud Platform account with billing enabled
- `gcloud` CLI installed and configured
- Terraform installed (for automation lab)
- Basic knowledge of GCP networking concepts

## Project Structure

```
├── README.md
├── docs/
│   ├── lab1-loadbalancer-guide.md
│   └── lab2-terraform-guide.md
├── scripts/
│   ├── setup-firewall-rules.sh
│   ├── create-instance-templates.sh
│   ├── deploy-load-balancer.sh
│   ├── stress-test.sh
│   └── cleanup.sh
├── terraform/
│   ├── provider.tf
│   ├── managementnet.tf
│   ├── privatenet.tf
│   ├── mynetwork.tf
│   └── instance/
│       └── main.tf
├── configs/
│   ├── cloud-armor-policy.json
│   └── startup-script.sh
└── monitoring/
    └── load-balancer-dashboard.json
```

## Lab 1: Application Load Balancer with Cloud Armor

### Objectives

- Create HTTP and health check firewall rules
- Configure instance templates and managed instance groups
- Deploy Application Load Balancer with IPv4 and IPv6
- Implement stress testing
- Configure Cloud Armor security policies

### Quick Start

1. Clone this repository
2. Follow the [detailed lab guide](docs/lab1-loadbalancer-guide.md)
3. Execute the setup scripts in order
4. Test and verify the deployment

### Key Commands

```bash
# Set up firewall rules
./scripts/setup-firewall-rules.sh

# Create instance templates
./scripts/create-instance-templates.sh

# Deploy load balancer
./scripts/deploy-load-balancer.sh

# Run stress test
./scripts/stress-test.sh
```

## Lab 2: Terraform Network Automation

### Objectives

- Create custom and auto-mode networks using Terraform
- Implement reusable instance modules
- Deploy multi-region network infrastructure
- Verify connectivity between networks

### Quick Start

1. Navigate to the `terraform/` directory
2. Follow the [Terraform lab guide](docs/lab2-terraform-guide.md)
3. Initialize and apply Terraform configurations

### Key Commands

```bash
cd terraform/
terraform init
terraform plan
terraform apply
```

## Getting Started

### Option 1: Manual Deployment (Lab 1)

1. **Clone the repository**:
   ```bash
   git clone <your-repo-url>
   cd gcp-loadbalancer-cloudarmor
   ```

2. **Set up your GCP environment**:
   ```bash
   gcloud config set project YOUR_PROJECT_ID
   gcloud auth login
   ```

3. **Follow the step-by-step guide**:
   - See `docs/lab1-loadbalancer-guide.md` for detailed instructions

### Option 2: Terraform Automation (Lab 2)

1. **Navigate to Terraform directory**:
   ```bash
   cd terraform/
   ```

2. **Initialize Terraform**:
   ```bash
   terraform init
   ```

3. **Deploy infrastructure**:
   ```bash
   terraform plan
   terraform apply
   ```

## Key Learning Outcomes

After completing this project, you will understand:

- Google Cloud Application Load Balancing concepts
- Multi-region backend configuration
- Cloud Armor security implementation
- Infrastructure as Code with Terraform
- Network security best practices
- Load balancer monitoring and troubleshooting

## Monitoring and Observability

The project includes:

- Health check monitoring
- Load balancer metrics
- Cloud Armor policy logs
- Traffic distribution analysis
- Performance benchmarking

## Security Features

- **Cloud Armor Integration**: IP allowlist/denylist capabilities
- **Firewall Rules**: Granular traffic control
- **Network Segmentation**: Isolated VPC networks
- **SSL/TLS Termination**: Secure traffic handling

## Cleanup

To avoid ongoing charges:

```bash
# For manual deployment
./scripts/cleanup.sh

# For Terraform deployment
cd terraform/
terraform destroy
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## Troubleshooting

Common issues and solutions:

- **502/404 Errors**: Wait 5-10 minutes for load balancer provisioning
- **Health Check Failures**: Verify firewall rules and instance startup
- **Terraform Errors**: Check provider version and resource dependencies

## Additional Resources

- [Google Cloud Load Balancing Documentation](https://cloud.google.com/load-balancing/docs)
- [Cloud Armor Documentation](https://cloud.google.com/armor/docs)
- [Terraform Google Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs)

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

**Note**: This project is for educational purposes and demonstrates GCP networking and security concepts. Always follow your organization's security policies and best practices in production environments.
