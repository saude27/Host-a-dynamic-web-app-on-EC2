![Alt text](/nest-ref-ach3.png)

![Alt text](/nest-ref-ach2.png)

![Alt text](/nest-ref-ach.png)

![Alt text](/assignment4-ref-ach.png)


# Dynamic Website Hosting on AWS

This project demonstrates the deployment and hosting of a dynamic website on AWS, utilizing various services and components to ensure high availability, scalability, security, and fault tolerance.

## Architecture Overview

The website is hosted on EC2 instances within a Virtual Private Cloud (VPC) configured with public and private subnets spanning two Availability Zones. The infrastructure leverages the following AWS resources:

- **Virtual Private Cloud (VPC)**: A logically isolated section of the AWS cloud where AWS resources are launched.
- **Internet Gateway**: Enables communication between the VPC instances and the internet.
- **Security Groups**: Act as virtual firewalls to control inbound and outbound traffic.
- **Availability Zones**: Multiple Availability Zones are used to increase reliability and fault tolerance.
- **Public Subnets**: Host infrastructure components like the NAT Gateway and Application Load Balancer.
- **Private Subnets**: Host the web servers (EC2 instances) for enhanced security.
- **NAT Gateway**: Allows instances in private subnets to access the internet.
- **EC2 Instance Connect Endpoint**: Enables secure connections to EC2 instances within both public and private subnets.
- **Application Load Balancer**: Distributes incoming web traffic across multiple EC2 instances in an Auto Scaling group.
- **Auto Scaling Group**: Automatically manages the EC2 instances hosting the website, ensuring availability, scalability, fault tolerance, and elasticity.
- **AWS Certificate Manager**: Secures application communications with SSL/TLS certificates.
- **Simple Notification Service (SNS)**: Sends notifications about activities within the Auto Scaling Group.
- **Route 53**: Provides DNS services for registering and managing the website's domain name.
- **S3 Bucket**: Stores the application code and assets.
- **Amazon Machine Image (AMI)**: A pre-configured virtual machine image that serves as a blueprint for launching instances in the AWS Elastic Compute Cloud (EC2).

## Deployment

The deployment of this infrastructure is automated using scripts and configuration files stored in an S3 bucket and a GitHub repository. The repository includes the following:

- **Reference Diagram**: A visual representation of the AWS infrastructure and its components.
- **Deployment Scripts**: Scripts to provision and configure the necessary AWS resources.

## Usage

1. Deploy the website code and assets to the appropriate S3 bucket.
2. Access the website using the registered domain name.

## Deployment Script

```bash
#!/bin/bash

# Update all the packages on the server to their latest versions
sudo yum update -y

# Install Apache web server, enable it to start on boot, and start it immediately
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP along with necessary extensions
sudo yum install php -y
sudo dnf install -y php-cli php-fpm php-mysqlnd php-bcmath php-ctype php-fileinfo php-json php-mbstring php-openssl php-pdo php-gd php-tokenizer php-xml php-curl

# Install MySQL version 8
sudo dnf install -y https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Install php-curl and restart Apache server
php -m | grep curl
sudo dnf install -y php-curl
sudo systemctl restart httpd

# Update PHP configuration in php.ini
sudo nano /etc/php.ini
# Update the following settings:
# i. memory_limit = 128M
# ii. max_execution_time = 300

# Restart Apache server
sudo systemctl restart httpd

# Enable the 'mod_rewrite' module in Apache
sudo sed -i '/<Directory "/var/www/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# Set environment variable for S3 bucket name
S3_BUCKET_NAME=my-dynamic-web-bucket

# Sync website code from S3 bucket to /var/www/html
sudo aws s3 sync s3://"$S3_BUCKET_NAME" /var/www/html

# Change working directory to /var/www/html
cd /var/www/html

# Extract application code and configure
sudo unzip nest-app.zip
sudo cp -R nest-app/. /var/www/html/
sudo rm -rf nest-app nest-app.zip

# Set permissions for the '/var/www/html' directory and 'storage/'
sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/

# Update database credentials in .env file
sudo vi .env

# Restart Apache server
sudo service httpd restart
```


