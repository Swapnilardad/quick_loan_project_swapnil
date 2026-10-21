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

