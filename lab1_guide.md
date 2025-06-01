# Lab 1: Application Load Balancer with Cloud Armor

## Overview

This lab guides you through configuring an Application Load Balancer with global backends and implementing Cloud Armor security policies. You'll learn to stress test the load balancer and implement IP denylisting for security.

## Architecture Diagram

```
Internet → Global Load Balancer → Backend Services
                ↓
        [Region 1 MIG] ← Cloud Armor → [Region 2 MIG]
```

## Prerequisites

- Google Cloud Project with billing enabled
- Compute Engine API enabled
- Sufficient IAM permissions

## Task 1: Configure HTTP and Health Check Firewall Rules

### Create HTTP Firewall Rule

1. Navigate to **VPC Network > Firewall**
2. Click **Create Firewall Rule**
3. Configure the following:

| Property | Value |
|----------|--------|
| Name | `default-allow-http` |
| Network | `default` |
| Targets | `Specified target tags` |
| Target tags | `http-server` |
| Source filter | `IPv4 Ranges` |
| Source IPv4 ranges | `0.0.0.0/0` |
| Protocols and ports | `TCP: 80` |

### Create Health Check Firewall Rule

Health checks come from IP ranges: `130.211.0.0/22` and `35.191.0.0/16`

1. Click **Create Firewall Rule**
2. Configure:

| Property | Value |
|----------|--------|
| Name | `default-allow-health-check` |
| Network | `default` |
| Targets | `Specified target tags` |
| Target tags | `http-server` |
| Source filter | `IPv4 Ranges` |
| Source IPv4 ranges | `130.211.0.0/22, 35.191.0.0/16` |
| Protocols and ports | `TCP` |

## Task 2: Configure Instance Templates and Create Instance Groups

### Create Instance Templates

#### Region 1 Template

1. Navigate to **Compute Engine > Instance Templates**
2. Click **Create Instance Template**
3. Configure:

| Property | Value |
|----------|--------|
| Name | `region1-template` |
| Location | `Global` |
| Series | `E2` |
| Machine Type | `e2-micro` |
| Network tags | `http-server` |
| Subnetwork | `default (Region 1)` |

4. Under **Management > Metadata**, add:
   - Key: `startup-script-url`
   - Value: `gs://cloud-training/gcpnet/httplb/startup.sh`

#### Region 2 Template

1. Copy Region 1 template
2. Change name to `region2-template`
3. Change subnetwork to `default (Region 2)`

### Create Managed Instance Groups

#### Region 1 MIG

1. Navigate to **Compute Engine > Instance Groups**
2. Click **Create Instance Group**
3. Configure:

| Property | Value |
|----------|--------|
| Name | `region1-mig` |
| Instance template | `region1-template` |
| Location | `Multiple zones` |
| Region | `Region 1` |
| Min instances | `1` |
| Max instances | `2` |
| CPU utilization | `80%` |
| Initialization period | `45 seconds` |

#### Region 2 MIG

Repeat for Region 2 with `region2-mig` and `region2-template`.

## Task 3: Configure the Application Load Balancer

### Start Configuration

1. Navigate to **Network Services > Load Balancing**
2. Click **Create Load Balancer**
3. Select **Application Load Balancer HTTP(S)**
4. Choose **Public facing (external)**
5. Select **Best for global workloads**
6. Choose **Global external Application Load Balancer**

### Configure Frontend

Set up both IPv4 and IPv6 frontends:

**IPv4 Frontend:**
- Protocol: `HTTP`
- IP version: `IPv4`
- IP address: `Ephemeral`
- Port: `80`

**IPv6 Frontend:**
- Protocol: `HTTP`
- IP version: `IPv6`
- IP address: `Auto-allocate`
- Port: `80`

### Configure Backend

1. Create backend service named `http-backend`
2. Add Region 1 backend:
   - Instance group: `region1-mig`
   - Port: `80`
   - Balancing mode: `Rate`
   - Maximum RPS: `50`

3. Add Region 2 backend:
   - Instance group: `region2-mig`
   - Port: `80`
   - Balancing mode: `Utilization`
   - Maximum utilization: `80%`

4. Create health check:
   - Name: `http-health-check`
   - Protocol: `TCP`
   - Port: `80`

5. Enable logging with 100% sample rate

## Task 4: Test the Application Load Balancer

### Access the Load Balancer

1. Note the IPv4 and IPv6 addresses from the load balancer details
2. Access via browser: `http://[LB_IP_v4]`
3. Verify you see backend information (hostname, client IP, server location)

### Stress Test Setup

1. Create a VM in Region 3 for testing:

```bash
# Create siege VM
gcloud compute instances create siege-vm \
  --zone=ZONE_3 \
  --machine-type=e2-standard-2 \
  --image-family=debian-11 \
  --image-project=debian-cloud
```

2. SSH into siege-vm and install siege:

```bash
sudo apt-get update
sudo apt-get -y install siege
```

3. Set environment variable:

```bash
export LB_IP=[YOUR_LB_IPv4_ADDRESS]
```

4. Run stress test:

```bash
siege -c 150 -t120s http://$LB_IP
```

### Monitor Load Distribution

1. Navigate to **Load Balancing > Backends**
2. Click on `http-backend`
3. Monitor the **Monitoring** tab
4. Observe traffic distribution between regions

## Task 5: Implement Cloud Armor Security Policy

### Create Security Policy

1. Navigate to **Network Security > Cloud Armor Policies**
2. Click **Create Policy**
3. Configure:

| Property | Value |
|----------|--------|
| Name | `denylist-siege` |
| Default rule action | `Allow` |

### Add Deny Rule

1. Click **Add Rule**
2. Configure:

| Property | Value |
|----------|--------|
| Condition | `[SIEGE_VM_EXTERNAL_IP]` |
| Action | `Deny` |
| Response code | `403 (Forbidden)` |
| Priority | `1000` |

3. Apply to backend service `http-backend`

### Verify Security Policy

1. Test from siege-vm:

```bash
curl http://$LB_IP
```

Expected result: `403 Forbidden`

2. Test from your browser - should still work due to default allow rule

### Monitor Security Logs

1. Navigate to **Cloud Armor Policies > denylist-siege > Logs**
2. Click **View Policy Logs**
3. Filter by Application Load Balancer
4. Examine blocked requests in Query Results

## Key Concepts Learned

### Load Balancing
- **Global Load Balancing**: Traffic routed to closest healthy backend
- **Backend Configuration**: Different balancing modes (RPS vs Utilization)
- **Health Checks**: Automatic traffic routing based on instance health
- **IPv4/IPv6 Support**: Dual-stack load balancing capabilities

### Cloud Armor
- **Security Policies**: Rule-based traffic filtering
- **IP Allowlisting/Denylisting**: Granular access control
- **Edge Security**: Protection at Google's network edge
- **Logging**: Comprehensive audit trail for security events

### Autoscaling
- **Managed Instance Groups**: Automatic scaling based on metrics
- **CPU Utilization**: Scale based on backend load
- **Regional Distribution**: Traffic balanced across multiple regions

## Troubleshooting Tips

### Common Issues

1. **502/404 Errors**: 
   - Solution: Wait 5-10 minutes for full provisioning
   - Check health check configuration

2. **Health Check Failures**:
   - Verify firewall rules allow health check IP ranges
   - Ensure startup script runs successfully

3. **Cloud Armor Not Working**:
   - Verify policy is attached to correct backend service
   - Allow 2-3 minutes for policy propagation

4. **Uneven Load Distribution**:
   - Check backend capacity settings
   - Verify autoscaling configuration

### Verification Commands

```bash
# Check instance health
gcloud compute backend-services get-health http-backend --global

# View load balancer details
gcloud compute forwarding-rules list

# Check Cloud Armor policies
gcloud compute security-policies list
```

## Cleanup Instructions

To avoid ongoing charges, delete resources in this order:

1. **Load Balancer**: Delete forwarding rules and backend services
2. **Instance Groups**: Delete managed instance groups
3. **Instance Templates**: Delete templates
4. **Cloud Armor**: Delete security policies
5. **Firewall Rules**: Delete custom firewall rules
6. **VM Instances**: Delete siege-vm

```bash
# Example cleanup commands
gcloud compute forwarding-rules delete http-lb-forwarding-rule --global
gcloud compute backend-services delete http-backend --global
gcloud compute instance-groups managed delete region1-mig --region=REGION1
gcloud compute instance-groups managed delete region2-mig --region=REGION2
```

## Next Steps

- Explore SSL/TLS termination
- Implement custom health checks
- Configure URL mapping for path-based routing
- Set up monitoring and alerting
- Explore Cloud CDN integration

---

This completes Lab 1 of the Application Load Balancer with Cloud Armor implementation.