**AWS Auto Scaling in Linux - Lab Guide**
This guide explains how to use AWS Auto Scaling to create an Amazon EC2 instance and configure it to automatically scale based on demand. This lab uses AWS CLI to manage instances and deploy an Auto Scaling group.

**Objectives**
By the end of this lab, you will:

- Create an EC2 instance using AWS CLI.
- Create a new AMI for Auto Scaling.
- Configure AWS CLI.
- Create an Auto Scaling Group.
- Create and configure an Application Load Balancer.
- Test and verify the Auto Scaling configuration.
- Task 1: Creating a New AMI for EC2 Auto Scaling

**Step 1.1: Connect to the Host Instance**
- Launch the AWS Management Console.
- Connect to an existing EC2 instance through EC2 Dashboard > Instances.
- Use EC2 Instance Connect to log in to the host instance.

**Step 1.2: Configure AWS CLI**
- Run the following command to configure AWS CLI:
- $aws configure
- Enter your:
-- AWS Access Key ID
-- AWS Secret Access Key
-- Default Region (e.g., us-west-2)
-- Output format (json)
  
**Step 1.3: Create a New EC2 Instance**
Use the following command to install required packages and set up a web server:
- sudo yum update -y
- sudo yum install httpd -y
- sudo systemctl start httpd
- sudo systemctl enable httpd
- echo "Welcome to the Web Server!" > /var/www/html/index.html

**Create an AMI for the EC2 instance:**
- aws ec2 create-image --instance-id <instance-id> --name "WebServer-AMI" --no-reboot

**Task 2: Creating an Auto Scaling Environment**

Step 2.1: Create an Application Load Balancer*
- Navigate to EC2 Dashboard > Load Balancers > Create Load Balancer.
- Choose Application Load Balancer and configure the following:
- Availability Zones: Select two public subnets.
- Security Group: Create a new security group with HTTP access enabled.
- Target Group: Create a new target group for the load balancer.

Step 2.2: Create a Launch Template*
- Go to EC2 Dashboard > Launch Templates > Create Launch Template.
- Configure the launch template:
- AMI ID: Use the AMI created in Task 1.
- Instance Type: t2.micro
- Security Group: Use the previously created security group.
- Save the template.

Step 2.3: Create an Auto Scaling Group*
- Go to EC2 Dashboard > Auto Scaling Groups > Create Auto Scaling Group.
- Configure the group:
-- Launch Template: Use the template created in Step 2.2.
-- VPC and Subnets: Select two private subnets.
-- Load Balancer: Attach the Application Load Balancer.
-- Scaling Policies: Use a target tracking policy.

**Task 3: Verifying and Testing the Auto Scaling Configuration**

Step 3.1: Verify Auto Scaling*
- Monitor the status of your Auto Scaling group in the EC2 Dashboard > Instances.
- Check that instances are launched across multiple Availability Zones.

Step 3.2: Test the Auto Scaling Configuration*
- Simulate high CPU usage to trigger scaling:
- sudo stress --cpu 2 --timeout 300
- Confirm that a new instance is launched.


