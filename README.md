CloudBridge Technologies — AWS Highly Available Web Application
CloudBridge Technologies is a responsive business website deployed on AWS. This project demonstrates a cloud-hosted web application using a custom VPC, multiple Availability Zones, an Auto Scaling Group (ASG), an Application Load Balancer (ALB), Amazon Route 53, AWS Certificate Manager (ACM), and Amazon CloudFront.
The goal is to make the website reachable through a custom domain while improving availability, traffic distribution, and secure HTTPS delivery.
Architecture Overview
Visitors
   |
   v
Custom Domain (registered with Go54)
   |
   v
Amazon Route 53 (DNS)
   |
   v
Amazon CloudFront (CDN / HTTPS edge delivery)
   |
   v
Application Load Balancer (ALB)
   |
   v
Target Group
   |
   v
Auto Scaling Group (ASG)
   |                         |
   v                         v
EC2 Instance 1             EC2 Instance 2
Public Subnet A            Public Subnet B
Availability Zone A        Availability Zone B

Custom VPC
├── Availability Zone A
│   ├── Public Subnet A — EC2 Instance 1
│   └── Additional subnet (as configured)
└── Availability Zone B
    ├── Public Subnet B — EC2 Instance 2
    └── Additional subnet (as configured)
Network layout: The custom VPC contains four subnets distributed across two Availability Zones. The two application instances are in Public Subnet A and Public Subnet B, respectively. The ALB is associated with subnets in both Availability Zones, and the ASG manages the EC2 capacity registered with the target group.
Note: The diagram reflects the architecture described for this project. Confirm the exact subnet names, CIDR blocks, routing, security-group rules, and whether CloudFront forwards to the ALB before publishing implementation screenshots or configuration details.

AWS Services and Technologies
- Amazon EC2: Runs the website on a CentOS server / CentOS-compatible Linux environment.
- Amazon VPC: Provides an isolated network with four subnets across two Availability Zones.
- Auto Scaling Group (ASG): Manages the EC2 instances and supports instance replacement and scaling according to the configured capacity and policies.
- Application Load Balancer (ALB): Receives application traffic and distributes requests to healthy registered targets.
- Target Group: Registers the EC2 instances and uses health checks to determine target health.
- Amazon Route 53: Hosts DNS records for the custom domain.
- AWS Certificate Manager (ACM): Provides the TLS certificate used for HTTPS. For a CloudFront viewer certificate, the ACM certificate must be in US East (N. Virginia), us-east-1.
- Amazon CloudFront: Delivers the site through AWS edge locations and can cache eligible content.
- Go54: Domain registrar used to register the custom domain.
- HTML, CSS and JavaScript: Front-end technologies used by the website.
Request Flow
1. A visitor enters the custom domain in a browser.
2. Route 53 resolves the domain to the CloudFront distribution using the configured DNS records.
3. CloudFront serves cached content when available and forwards eligible requests to its configured origin.
4. The origin is the ALB (if configured that way), which distributes requests to healthy targets in the target group.
5. The target group forwards requests to the EC2 instances managed by the ASG.
6. The application response travels back to the visitor.
Key Implementation Areas
1. Custom VPC and Subnets
- Created a custom VPC.
- Configured four subnets across two Availability Zones.
- Placed EC2 Instance 1 in Public Subnet A and EC2 Instance 2 in Public Subnet B.
- Configured subnet route tables and internet connectivity as required by the design.
2. EC2 and Web Server
- Launched CentOS-based EC2 instances.
- Installed and configured the web server and website files.
- Verified that the application responds on the expected web port.
3. Auto Scaling Group
- Configured an ASG to manage the application instances.
- Used a launch template or launch configuration to define instance settings.
- Connected the ASG to the target group.
- Set desired, minimum, and maximum capacity according to project requirements.
Actual scaling behavior depends on the capacity settings and scaling policies configured in AWS.
4. Application Load Balancer and Target Group
- Created an ALB spanning the selected Availability Zones.
- Created a target group for the application instances.
- Configured health checks and listener rules.
- Verified that healthy targets receive traffic.
5. Domain, Route 53, ACM and CloudFront
- Registered the domain through Go54.
- Created or configured a Route 53 hosted zone.
- Updated the domain's nameservers at Go54 to the Route 53 nameservers when using Route 53 as the authoritative DNS provider.
- Requested and validated an ACM certificate for HTTPS.
- Created a CloudFront distribution and configured its origin, viewer protocol policy, and certificate.
- Added Route 53 alias records for the domain and, if required, its www subdomain, pointing to the CloudFront distribution.
Important configuration detail: CloudFront and the ALB have separate TLS configuration. The certificate attached to CloudFront must be in us-east-1. If HTTPS is also configured between CloudFront and the ALB, the ALB needs a suitable certificate in the same AWS Region as the ALB.
Security Considerations
Review these settings before treating the deployment as production-ready:
- Allow inbound HTTP/HTTPS traffic to the ALB only as required.
- Prefer allowing application traffic to EC2 instances only from the ALB security group, rather than from all internet addresses.
- Restrict SSH access to trusted IP addresses or use a managed access method such as AWS Systems Manager where configured.
- Keep IAM permissions limited to the actions and resources required.
- Use HTTPS and redirect HTTP to HTTPS where appropriate.
- Keep the operating system, web server, and application dependencies patched.
- Avoid committing private keys, passwords, access keys, environment secrets, or sensitive configuration files to GitHub.
Because the two EC2 instances are in public subnets, ensure their security groups and routing rules are intentionally configured. A public subnet does not by itself make an instance safe or unsafe; exposure depends on routing and security-group rules.
Repository Contents
.
├── index.html
├── style.css
├── script.js
├── profile.jpeg
├── flyer1.jpeg
├── flyer2.jpeg
├── assets/
│   └── favicon.png
└── README.md
Testing and Validation Checklist
- [ ] The website loads on each EC2 instance.
- [ ] Both EC2 instances register as healthy in the target group.
- [ ] The ALB DNS name serves the application.
- [ ] The ASG shows the expected desired capacity and healthy instances.
- [ ] Route 53 records resolve the custom domain to CloudFront.
- [ ] The domain's HTTPS certificate is valid.
- [ ] HTTP redirects to HTTPS if that behavior is intended.
- [ ] CloudFront reaches the configured origin successfully.
- [ ] The site remains available if one registered target becomes unhealthy.
What I Learned
This project provided practical experience with AWS networking, Linux-based web hosting, load balancing, target health checks, auto scaling, DNS configuration, TLS certificates, and content delivery through CloudFront. It also demonstrates how individual AWS services work together to deliver a website through a custom domain.
Future Improvements
- Move application instances into private subnets and use a suitable outbound-access design, if the architecture and project requirements allow.
- Add CloudWatch alarms, dashboards, and centralized logs.
- Configure a scaling policy based on suitable metrics.
- Automate infrastructure provisioning with AWS CloudFormation or Terraform.
- Add a deployment pipeline for repeatable releases.
- Document actual CIDR blocks, listeners, health-check paths, scaling limits, and security-group rules.
Deployment Notes
This README describes the architecture at a high level. Add screenshots of the VPC/subnets, ASG, ALB target health, CloudFront distribution, Route 53 records, and ACM certificate status to document the actual deployment. Do not publish account IDs, private keys, credentials, or other sensitive values.
