# quick_loan_project_swapnil
Project Overview
QuickLoan is a cloud-hosted loan application platform. Customers browse loan products (Home, Gold, Vehicle
and Personal loans) and submit an application form. The application runs on Amazon EC2, static images are
served from Amazon S3, customer data is stored in Amazon RDS MySQL, and the infrastructure is built to scale
and stay available.
Layer T
Architecture Flow Chart
A visitor's browser resolves the domain through GoDaddy name servers delegated to Route 53. The DNS record
points to the internet-facing Application Load Balancer, which forwards traffic to the Target Group. The Auto
Scaling Group keeps multiple Nginx + PHP web servers running across Availability Zones. The web servers read
and write customer data in a private RDS MySQL database and load images from S3. CloudWatch watches CPU
and notifies subscribers through SNS.
<img width="535" height="323" alt="Screenshot 2026-10-21 170240" src="https://github.com/user-attachments/assets/dc4c2bbc-f5c1-4f01-95c5-3738271baeb2" />

Step 1 - Networking and Security Groups A custom VPC was created to isolate all project resources. Public subnets host the internet-facing resources (Jump Server, web servers, load balancer) and a private subnet hosts the database. • VPC with public subnets in us-east-1a and us-east-1b, plus further subnets for the other Availability Zones used (us-east-1c for the load balancer and web servers, us-east-1d for the database). • Internet Gateway attached to the VPC; public route table sends 0.0.0.0/0 to the Internet Gateway. • NAT Gateway in a public subnet so private resources can fetch updates without being exposed. • Security group for app servers: SSH (22) from the admin's IP and HTTP (80) from anywhere. • Security group for the database: MySQL (3306) allowed only from the app-server security group.
Step 2 - Launch the EC2 Instances Three EC2 instances (all t3.micro) were launched: a Jump Server as the single SSH entry point, a Web Server for Nginx and PHP, and a Database server used as a MySQL client to reach RDS. • JUMPSERVER - us-east-1a (public subnet, reachable from the internet over SSH). • WEBSERVER - us-east-1b (hosts Nginx and the PHP application). • DATABASE - us-east-1d (used to connect to RDS and create the schema).

<img width="1366" height="768" alt="Screenshot (101)" src="https://github.com/user-attachments/assets/6c06f2a2-634b-4e56-bc70-d3b4bca5467b" />

Step 3 - Connect to the Servers through the Jump Server The private key files for the web and database servers were copied onto the Jump Server and locked down with chmod 400. From the Jump Server, SSH was used to reach the Web Server on its private IP (10.10.2.8) and the hostname was set to WEBSERVER. chmod 400 web.pem db.pem ssh -i web.pem ec2-user@10.10.2.8 sudo su hostnamectl set-hostname WEBSERVER

<img width="1366" height="768" alt="Screenshot (105)" src="https://github.com/user-attachments/assets/58a961bc-2f77-437d-bb75-ce0a51355f6e" />

