🚀 Data Migration: EC2 (MariaDB) → AWS RDS (MariaDB)

🧭 Objective

Migrate a MariaDB database from a Linux EC2 instance to an AWS RDS (MariaDB) instance — step by step, like a professional cloud engineer.

This guide simplifies the migration process with clear commands, visuals, and pro tips. Perfect for showcasing your AWS DevOps & Database migration skills.

🧩 Table of Contents

✅ Prerequisites

💻 Create & Configure EC2 (Linux)

🧱 Install MariaDB on EC2

🧑‍💼 Create Database & Employee Table

💾 Backup (mysqldump)

☁️ Setup AWS Networking for RDS

🗄️ Create RDS (MariaDB)

📤 Restore Backup into RDS

🔍 Verify Migration

🧹 Cleanup & Tips


✅ 1. Prerequisites

Before starting, ensure you have:

AWS account with EC2 and RDS permissions.

AWS CLI configured → aws configure

SSH key pair to access your EC2.

Basic understanding of MySQL/MariaDB.

💡 Pro Tip: Always keep your DB credentials secure using AWS Secrets Manager.

💻 2. Create & Configure EC2 (Linux)
🧠 Step: Launch EC2 instance

AMI: Ubuntu Server 22.04 LTS (or Amazon Linux 2023)

Instance type: t3.micro (Free-tier eligible)

Ports: Allow SSH (22) from your IP

SSH into EC2:  ssh -i my-key.pem ubuntu@<EC2_PUBLIC_IP>


🔐 Configure Security Group

Allow inbound SSH (22) and later, allow RDS access (3306) from EC2.


🧱 3. Install MariaDB on EC2


sudo apt update && sudo apt install mariadb-server mariadb-client -y
sudo systemctl enable --now mariadb
sudo mysql_secure_installation

✅ Verify: sudo systemctl status mariadb

🧑‍💼 4. Create Database & Employee Table

Log in to MariaDB: sudo mysql -u root -p

Create Database, User & Table

CREATE DATABASE employee_db;
CREATE USER 'emp_user'@'%' IDENTIFIED BY 'EmpUserPassword123!';
GRANT ALL PRIVILEGES ON employee_db.* TO 'emp_user'@'%';
FLUSH PRIVILEGES;


USE employee_db;


CREATE TABLE employee (
id INT AUTO_INCREMENT PRIMARY KEY,
first_name VARCHAR(50),
last_name VARCHAR(50),
email VARCHAR(100),
department VARCHAR(50),
hire_date DATE,
salary DECIMAL(10,2)
);


INSERT INTO employee (first_name, last_name, email, department, hire_date, salary) VALUES
('Asha', 'Kumar', 'asha.kumar@example.com', 'Engineering', '2023-04-01', 55000.00),
('Ravi', 'Sharma', 'ravi.sharma@example.com', 'HR', '2022-09-15', 42000.00);

🧾 Check data: SELECT * FROM employee;

💾 5. Take Backup (mysqldump)

mysqldump -u root -p employee_db > employee_db_backup.sql

📂 Transfer to local or keep on EC2 for direct restore.

☁️ 6. Setup AWS Networking for RDS

Create:

VPC (if not already available)

DB Subnet Group (2+ private subnets)

Security Group allowing port 3306 inbound

aws ec2 create-security-group --group-name rds-sg --description "RDS MariaDB SG" --vpc-id <VPC_ID>
aws ec2 authorize-security-group-ingress --group-id <SG_ID> --protocol tcp --port 3306 --source-group <EC2_SG_ID>

🔒 Only allow trusted sources to connect (your EC2 or local IP).


🗄️ 7. Create RDS (MariaDB)

Use AWS Console or CLI:

aws rds create-db-instance \
--db-instance-identifier my-mariadb-rds \
--allocated-storage 20 \
--db-instance-class db.t3.micro \
--engine mariadb \
--master-username admin \
--master-user-password 'RdsAdminPass123!' \
--vpc-security-group-ids <RDS_SG_ID> \
--publicly-accessible true

Wait until status = available.

Retrieve endpoint:

aws rds describe-db-instances --db-instance-identifier my-mariadb-rds --query 'DBInstances[0].Endpoint.Address' --output text

📤 8. Restore Backup into RDS

💡 If connection fails, check RDS security group inbound rules and ensure EC2 can reach the endpoint.

🔍 9. Verify Migration

✅ You should see the same employee rows that were present on EC2.

🎯 Congratulations — Migration Complete!

🧹 10. Cleanup & Pro Tips

Terminate EC2 if no longer needed.

Restrict or delete RDS security rules.

Store backups in Amazon S3 for durability.

Automate using AWS DMS or CloudFormation for large-scale setups.

⚙️ Pro Tip: Use AWS DMS (Database Migration Service) for production-grade replication and continuous syncs.

📘 Summary Flow Diagram

graph TD;
A[EC2: MariaDB Source] -->|mysqldump| B[Backup SQL File];
B -->|mysql import| C[RDS: MariaDB Destination];
A --> D[Security Group: SSH 22];
C --> E[Security Group: MySQL 3306];
E -->|Connected| A;

Author: Arkan Tandel 🧠
email : arkantandel@gmal.com
Project: Data Migration from EC2 → RDS
GitHub: https://github.com/arkantandel

🌟 “Migration isn’t magic — it’s mastering the flow of data.”





