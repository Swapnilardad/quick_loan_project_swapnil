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


Step 4 - Upload the Website Files with WinSCP The website folders (includes, nginx, public) were transferred from the local machine to the Web Server using WinSCP over SFTP (port 22, user ec2-user), then moved into the Nginx web root. mv /home/ec2-user/includes /usr/share/nginx/html mv /home/ec2-user/public   /usr/share/nginx/html chown -R nginx:nginx /usr/share/nginx/html/public chmod -R 755 /usr/share/nginx/html/public

<img width="1366" height="768" alt="Screenshot (108)" src="https://github.com/user-attachments/assets/a61dd9a2-a51b-4b7c-85bc-f2ff739a8265" />

Step 5 - Create the Amazon S3 Bucket for Images Static images were moved to Amazon S3 so the web servers do not depend on local storage. The bucket swapnil74245352 holds an images/ folder with the loan product pictures and the QuickLoan logo, which are made publicly readable. • Bucket: swapnil74245352 (us-east-1), folder images/ with 5 objects. • Files include gold_loan.jpg, home_loan.jpg, personal_loan.jpg, quickloan_logo.png and the vehicle loan image.


<img width="1366" height="768" alt="Screenshot (112)" src="https://github.com/user-attachments/assets/426555de-07ae-41ec-84d3-d692fbf88dc4" />

Step 6 - Update the Image URLs in the Application index.html was edited with vim so every image points to the new S3 bucket instead of the old one. The example below matches the pattern visible in the screenshot. vim /usr/share/nginx/html/public/index.html <img src="https://swapnil74245352.s3.us-east-1.amazonaws.com/images/personal_loan.jpg"     alt="Personal Loan">

<img width="1366" height="768" alt="Screenshot (113)" src="https://github.com/user-attachments/assets/c0a981cf-0fe2-439b-a4fd-6132adf7d563" />

Step 7 - Create the Amazon RDS MySQL Database A managed MySQL database named my-project-db was created in the same VPC, with private access and the database security group attached so only the app servers can connect. The status shows Creating while the instance launches. • DB identifier: my-project-db, engine: MySQL Community, Region/AZ: us-east-1d. • The endpoint is then placed in includes/db_connect.php as the database host (keep the username and password out of the document and out of GitHub).

<img width="1366" height="768" alt="Screenshot (115)" src="https://github.com/user-attachments/assets/f1fdf313-b3a5-482c-9d49-75e4a9141965" />

Step 8 - Configure the Nginx Virtual Host A virtual host file (quickloan.conf) was created on the Web Server, moved to /etc/nginx/conf.d/, validated with nginx -t, and Nginx was restarted. The configuration test reported success. vim /home/ec2-user/nginx/quickloan.conf mv /home/ec2-user/nginx/quickloan.conf /etc/nginx/conf.d nginx -t systemctl restart nginx

<img width="1366" height="768" alt="Screenshot (122)" src="https://github.com/user-attachments/assets/a83f8fcd-89b7-429f-bec8-4bb249683a6c" />

Step 9 - Test the Application The site was opened in a browser using the server's public IP. The home page shows the QuickLoan logo, the hero banner and the four loan products with images served from S3. The Apply for Loan page shows the application form (Name, Contact Number, Email Address and Type of Loan).

<img width="1366" height="768" alt="Screenshot (123)" src="https://github.com/user-attachments/assets/854d793c-aa98-4f57-9313-7a6318843eb4" />

<img width="1366" height="768" alt="Screenshot (124)" src="https://github.com/user-attachments/assets/94d337cb-938f-40be-93fa-a54b25840267" />

Step 10 - Create the Database Table and Verify the Data From the DATABASE server, the MySQL client connected to RDS. The quickloan_db database holds the applications table. After submitting the form, the rows were verified with a SELECT query. • Columns: id (INT, primary key, auto_increment), name, contact, email and loan_type. mysql -h <RDS-ENDPOINT> -u admin -p USE quickloan_db; DESC applications; SELECT * FROM applications;

<img width="1366" height="768" alt="Screenshot (125)" src="https://github.com/user-attachments/assets/fb34c929-d693-41df-86f9-835ce83144df" />


Step 11 - Create the Target Group A target group was created so the load balancer knows where to send traffic. The web server instances are registered as targets and health-checked on the root path. • Name: MY-PROJ-TG, target type: Instance. • Protocol / port: HTTP : 80 (HTTP1), IP type IPv4. • Health check: HTTP, path /, port traffic-port, interval 30 seconds.


<img width="1366" height="768" alt="Screenshot (126)" src="https://github.com/user-attachments/assets/9ec519fb-83fc-4662-ae51-3e5a73b8a69a" />

Step 12 - Create the Application Load Balancer An internet-facing Application Load Balancer was created and attached to the target group. It spans two Availability Zones (us-east-1b and us-east-1c) so traffic keeps flowing if one zone fails. • Name: MY-PROJ-LB, type: Application, scheme: Internet-facing, IPv4. • Listener: HTTP 80 forwarding to MY-PROJ-TG.


<img width="1366" height="768" alt="Screenshot (128)" src="https://github.com/user-attachments/assets/655ebe7f-e68d-4de1-878b-f36fcf6f6c94" />

Step 13 - Configure the Domain with Route 53 and GoDaddy The domain swapnilardad.in was bought on GoDaddy. A public hosted zone was created in Route 53, a CNAME record for www was pointed to the load balancer's DNS name, and GoDaddy's name servers were replaced with the four Route 53 name servers. • Route 53 hosted zone contains NS, SOA and a CNAME record (www) that points to the MY-PROJ-LB DNS name (TTL 300). • In GoDaddy, I'll use my own nameservers was selected and the AWS name servers were entered.


<img width="1366" height="768" alt="Screenshot (129)" src="https://github.com/user-attachments/assets/7a1b0dd6-df66-4eb3-8cad-2100f966ddee" />


Step 14 - Create the AMI, Launch Template and Auto Scaling Group An AMI was created from the fully working web server and used in a Launch Template (MY-PROJECT-TEMPLATE). The Auto Scaling Group MY-PROJ-AUTOSCALING uses that template, is attached to the load balancer, and starts with a desired capacity of 2. The group shows Updating capacity while it launches the instances.

<img width="1366" height="768" alt="Screenshot (135)" src="https://github.com/user-attachments/assets/eae4999f-850a-4e25-b9a3-55074afe6877" />

<img width="1366" height="768" alt="Screenshot (136)" src="https://github.com/user-attachments/assets/fa1ae02b-12b3-4110-96ae-a4312623e4c8" />

Step 15 - Create the SNS Topic and Email Subscription An SNS topic was created for alerts and an email address was subscribed to it. The subscription stays in Pending confirmation until the recipient clicks the confirmation link in the email AWS sends. • Topic: MY-PROJ-TOPIC, type Standard, display name ALERT. • Subscription protocol: EMAIL, status: Pending confirmation.

<img width="1366" height="768" alt="Screenshot (134)" src="https://github.com/user-attachments/assets/2f47a376-70bd-481f-8806-a62a8511fa29" />

Step 16 - Stress Test and CloudWatch Alarm To prove monitoring and scaling work, the stress tool was installed on a web server and run to push CPU usage to 100%. The CloudWatch alarm named CPU > 90% watches CPUUtilization and triggers when it stays above 90 for one data point within five minutes, which sends an SNS notification and lets Auto Scaling react. yum install -y stress stress -c 90


<img width="1366" height="768" alt="Screenshot (138)" src="https://github.com/user-attachments/assets/8014940f-cc33-4ce0-b3cc-e87762f60996" />

<img width="1366" height="768" alt="Screenshot (141)" src="https://github.com/user-attachments/assets/f277ff78-1b59-4a3d-99c2-7f6106e6b802" />


Architecture Outcome The final deployment provides: • Network isolation with a custom VPC and public / private subnet separation. • Controlled administrative access through a Jump Server. • A scalable application layer: Launch Template + Auto Scaling Group (desired 2) behind an Application Load Balancer. • High availability by spreading instances and load balancer nodes across multiple Availability Zones. • A managed MySQL database on Amazon RDS reachable only from the app servers. • Static content stored on Amazon S3. • A custom domain through Route 53 and monitoring with CloudWatch and SNS. Recommended improvements • Do not keep database passwords in db_connect.php; use AWS Secrets Manager or environment variables. • Place RDS in a DB subnet group that spans at least two Availability Zones, and enable automated backups for anything beyond a lab. • Move web servers into private subnets behind the load balancer and use HTTPS (ACM certificate) on the listener. • Remove personal data from screenshots before sharing publicly.
