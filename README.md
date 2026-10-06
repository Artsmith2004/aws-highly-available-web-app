# Highly Available Web Application on AWS

Designed and deployed a **fault-tolerant, auto-scaling web application** on AWS, with a load balancer distributing traffic across multiple EC2 instances and automated **CloudWatch + SNS** email alerts to support high availability.

## Tools & Services

| Service | Purpose |
|---|---|
| **Amazon EC2** | Hosts the Apache web server |
| **AMI** | Reusable image of the fully configured web server |
| **Launch Template** | Blueprint the Auto Scaling Group uses to launch instances |
| **Auto Scaling Group (ASG)** | Keeps the desired number of healthy instances running |
| **Classic Load Balancer** | Distributes incoming traffic across instances |
| **CloudWatch** | Monitors CPU utilization and triggers alarms |
| **SNS** | Sends email notifications when an alarm fires |

## Architecture

```
                    Users (Internet)
                          |
                Classic Load Balancer
                          |
        +-----------------+-----------------+
        |                 |                 |
      EC2 (AMI)         EC2 (AMI)         EC2 (AMI)      <- Auto Scaling Group (min 2, max 5)
        |
  CloudWatch Alarm (CPUUtilization)
        |
   SNS Topic  -->  Email notification
```

## Implementation

### Step 1: Launch an EC2 instance

Launched a `t3.micro` instance (Amazon Linux 2023) in the `ap-south-1` (Mumbai) region.

![EC2 instances](images/01-ec2-instances.png)

![EC2 instance details](images/02-ec2-instance-details.png)

### Step 2: Install and start Apache

Connected using EC2 Instance Connect and set up the Apache web server:

```bash
sudo su -
sudo yum install httpd -y
sudo systemctl enable httpd
sudo systemctl start httpd
sudo systemctl status httpd
```

![Apache installation](images/03-httpd-install.png)

### Step 3: Deploy the website

Downloaded the Clarity Bootstrap template into the Apache web root:

```bash
cd /var/www/html
wget https://bootstrapmade.com/clarity-bootstrap-agency-template/
```

![Deploying the website](images/04-deploy-website.png)

### Step 4: Create an AMI

Created an AMI named `janu` from the configured instance so that every new instance starts with Apache and the website already in place. AWS automatically created the backing EBS snapshot and volumes.

![AMI](images/05-ami.png)

![Snapshot](images/06-snapshot.png)

![EBS volumes](images/07-volumes.png)

### Step 5: Create a Launch Template

The launch template uses the custom AMI with instance type `t3.micro`, a key pair and a security group.

![Launch templates](images/08-launch-templates.png)

![Launch template details](images/09-launch-template-details.png)

### Step 6: Create the Auto Scaling Group

| Setting | Value |
|---|---|
| Desired capacity | 5 |
| Scaling limits | Min 2 – Max 5 |
| Launch template | `janu` (version Default) |
| Status | At desired capacity, 5/5 healthy |

If an instance becomes unhealthy, the ASG automatically replaces it.

![Auto Scaling groups](images/10-asg-list.png)

![ASG details](images/11-asg-details.png)

### Step 7: Configure the Classic Load Balancer

Created an internet-facing **Classic Load Balancer** (`janu`) across 3 Availability Zones. All instances launched by the ASG are registered with it, and the load balancer reported **6 of 6 instances in service**.

![Classic Load Balancer](images/12-classic-load-balancer.png)

### Step 8: CloudWatch alarm and SNS notification

Created a CloudWatch alarm on the `CPUUtilization` metric.

![Select metric](images/13-select-metric.png)

![CPUUtilization metric](images/14-cpu-metric.png)

Configured the alarm to notify an **SNS topic** (email subscription) when the alarm state is `In alarm`.

![SNS notification action](images/15-sns-action.png)

The alarm uses the condition `CPUUtilization < 20` for 1 datapoint within 5 minutes. This low-threshold condition was used to test and verify that the alarm and email alert work.

![Alarm list](images/16-alarms.png)

![Alarm graph](images/17-alarm-graph.png)

### Step 9: Final result

The website opens successfully through the **Load Balancer DNS name**, confirming that traffic is routed to the Auto Scaling Group instances.

![Final result via Load Balancer](images/18-final-result.png)

## Key Highlights

- **High availability:** multiple instances across Availability Zones behind a load balancer
- **Fault tolerance:** unhealthy instances are automatically replaced by the ASG
- **Scalability:** capacity can scale between 2 and 5 instances
- **Monitoring & alerting:** CloudWatch alarm with SNS email notifications
- **Consistency:** AMI + Launch Template ensure identical instances

## What I Learned

- Creating custom AMIs and launch templates
- Configuring Auto Scaling Groups and health checks
- Load balancing traffic across EC2 instances
- Setting up CloudWatch alarms and SNS notifications

## Clean-up

To avoid unnecessary charges, delete the Auto Scaling Group, Load Balancer, EC2 instances, AMI (and its snapshot) and CloudWatch alarm after testing.

## Author

**Janani**
