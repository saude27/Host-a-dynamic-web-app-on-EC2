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
- **Amazon Machine Image (AMI) is a pre-configured virtual machine image that serves as a blueprint for launching instances in the AWS Elastic Compute Cloud (EC2)
## Deployment

The deployment of this infrastructure is automated using scripts and configuration files stored in S3 bucket and GitHub repository. The repository includes the following:

- **Reference Diagram**: A visual representation of the AWS infrastructure and its components.
- **Deployment Scripts**: Scripts to provision and configure the necessary AWS resources.

## Usage
1. Deploy the website code and assets to the appropriate S3 bucket.
2. Access the website using the registered domain name.

 Deployment Script

# This command indicates that the script should be interpreted and executed using the Bash shell
#!/bin/bash

# This command updates all the packages on the server to their latest versions
sudo yum update -y


# This series of commands installs the Apache web server, enables it to start on boot, and then starts the server immediately

sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd


# This command installs PHP along with several necessary extensions for the application to run
sudo yum install php -y
sudo dnf install -y php-cli php-fpm php-mysqlnd php-bcmath php-ctype php-fileinfo php-json php-mbstring php-openssl php-pdo php-gd php-tokenizer php-xml php-curl



## These commands Installs MySQL version 8
# Install the MySQL Community repository
sudo dnf install -y https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm


# Install the MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server


# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# install php-curl
php -m | grep curl
sudo dnf install -y php-curl
sudo systemctl restart httpd


# Open your php configuration file php.ini and change the following settings.

i. memory_limit = 128M

ii. max_execution_time = 300
sudo nano /etc/php.ini

# Restart Service
sudo systemctl restart httpd

# This command enables the 'mod_rewrite' module in Apache on an EC2 Linux instance. It allows the use of .htaccess files for URL rewriting and other directives in the '/var/www/html' directory
sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf


# Environment Veriable
S3_BUCKET_NAME=my-dynamic-web-bucket

# This command downloads the contents of the specified S3 bucket to the '/var/www/html' directory on the EC2 instance
sudo aws s3 sync s3://"$S3_BUCKET_NAME" /var/www/html

# This command changes the current working directory to '/var/www/html', which is the standard directory for hosting web pages on a Unix-based server
cd /var/www/html

# This command is used to extract the contents of the application code zip file that was previously downloaded from the S3 bucket
sudo unzip nest-app.zip

# This command recursively copies all files, including hidden ones, from the 'nets-app' directory to the '/var/www/html/
sudo cp -R nest-app/. /var/www/html/

# This command permanently deletes the 'nest-app' directory and the 'nest-app' file.
sudo rm -rf nest-app nest-app.zip

# This command set permissions 777 for the '/var/www/html' directory and the 'storage/' directory

sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/

# This command will open the vi editor and allow you to edit the .env file to add your database credentials
sudo vi .env


# This command will restart the Apache server
sudo service httpd restart
