# Migrating an EC2-hosted Database to Amazon RDS
## Create an RDS instance that complies with these specifications.
```
Engine type: MariaDB
Templates: Dev/Test
DB instance identifier: <yourname?
Username: <admin>
Password: <yourpassword>
DB Instance Class: db.t3.micro
Storage type: General Purpose (SSD)
Allocated storage: 20 GiB
Subnet Group: subnet-group, where the database is not publicly accessible.
Create a security group, and unselect the default security group.
Availability Zone: Choose the first Availability Zone in the list, which ends in a. For example, if the Region is us-east-1, choose us-east-1a.
```
Database port: Keep the default TCP port of 3306.

<img width="1192" alt="Screenshot 2024-08-12 at 6 37 49 PM" src="https://github.com/user-attachments/assets/9bd6be7c-0937-47e5-b9db-64d518dc03a4">

## SSH into the instance database using System Manager Session Manager
<img width="1190" alt="Screenshot 2024-08-12 at 6 25 48 PM" src="https://github.com/user-attachments/assets/1c040a5a-2d82-49a9-ae01-67e7d382dc86">

## The Session manager output
I ran a few commands to show where the database on the ec2 instances was still functioning and running
```
service mariadb status
mysql --version
```
The below shows that the database on the mariadb was operational
<img width="1511" alt="Screenshot 2024-08-12 at 6 26 03 PM" src="https://github.com/user-attachments/assets/901ea6ba-dbd6-4244-97f5-799dbbb53742">

## Parameter store
This contains all the sensitive data about the database and the instance it runs on
<img width="1434" alt="Screenshot 2024-08-12 at 6 26 30 PM" src="https://github.com/user-attachments/assets/9b93b79c-72d6-4208-925c-8523cdaa60c5">

 ## Accessed the Mariadb database
 I used sql commands to log into the database and retrieve some of the data I was to transport to the RDS database. Below are some of the data I wanted to migrate to RDS
 ```
mysql -u root -p
```
This would prompt a password

I chose the /cafe/dbPassword parameter, and copied the Value to my clipboard. and entered into the space provided to enter password
<img width="835" alt="Screenshot 2024-08-12 at 6 29 14 PM" src="https://github.com/user-attachments/assets/737255ae-56bf-4142-85bb-7abe67cd2d0e">

<img width="609" alt="Screenshot 2024-08-12 at 6 57 07 PM" src="https://github.com/user-attachments/assets/96a04196-2e9d-42e3-97d1-a0571e1d07ac">

<img width="578" alt="Screenshot 2024-08-12 at 6 32 49 PM" src="https://github.com/user-attachments/assets/33f942af-452c-48f3-9061-de924e2613ff">

## Migrating database to a local file on the ec2 server using myqldump
```
mysqldump --databases cafe_db -u root -p > CafeDbDump.sql
```

<img width="808" alt="Screenshot 2024-08-12 at 6 35 33 PM" src="https://github.com/user-attachments/assets/f9e901b2-305b-45d8-ab2f-c644841fb998">

## Add EC2 security group to RDS security group
For the ec2 to send the database information to the RDS database, we would have to add an inbound rule to the RDS security group, to accept the traffic coming from the ec2 security group
<img width="1229" alt="Screenshot 2024-08-12 at 6 49 39 PM" src="https://github.com/user-attachments/assets/356dfb00-bcf4-458a-9d17-0bc01261fa7c">

## RDS Database
I was able to access the RDS database through the ec2 instance, and the output shows the current databases provisioned on the DB
<img width="1140" alt="Screenshot 2024-08-12 at 6 51 29 PM" src="https://github.com/user-attachments/assets/ab1b609b-6015-4b17-a95f-e0c8744ee8aa">

## Transfer sql database from e2 server to RDS DB
```
mysql -u admin -p --host <rds-endpoint> < CafeDbDump.sql
```
If you look closely, you can see the database was add called "cafe_db"

<img width="1166" alt="Screenshot 2024-08-12 at 6 56 15 PM" src="https://github.com/user-attachments/assets/5dd6b232-458d-48d9-9c89-01e05254ec77">

<img width="757" alt="Screenshot 2024-08-12 at 6 57 33 PM" src="https://github.com/user-attachments/assets/4b6fc994-f2b4-4462-bbc5-67ae17707592">

## Lets Access the website
<img width="1502" alt="Screenshot 2024-08-12 at 7 05 30 PM" src="https://github.com/user-attachments/assets/7a3d85f1-6805-404c-8f83-c50009700ce7">

## Order history, with the old database
<img width="1498" alt="Screenshot 2024-08-12 at 7 05 42 PM" src="https://github.com/user-attachments/assets/2d98f351-d3a0-4f24-a568-bb37ffe6e303">

## Changed Parameter store value with new credentials from RDS database
<img width="1425" alt="Screenshot 2024-08-12 at 7 32 19 PM" src="https://github.com/user-attachments/assets/6b422518-2e55-4cdf-bca3-2ba6175f7e72">

## Stopped MariaDB on ec2 instance
<img width="617" alt="Screenshot 2024-08-12 at 7 07 10 PM" src="https://github.com/user-attachments/assets/c48c0712-0995-4eaa-9d38-a99d44ef6ff8">

## Output to show its stopped 
<img width="1504" alt="Screenshot 2024-08-12 at 7 07 32 PM" src="https://github.com/user-attachments/assets/290bcc5b-a1bc-40c7-820a-fc50e4276b14">

## I made an order
This is to confirm if my newly configured RDS database is working
<img width="1501" alt="Screenshot 2024-08-12 at 7 08 05 PM" src="https://github.com/user-attachments/assets/c353983a-efe3-4a1a-9086-b04b68242f11">

## New Order display with older orders
This is proof that the database contains the orders from the mariadb run on the ec2 instance. Our transfer of data worked
<img width="1503" alt="Screenshot 2024-08-12 at 7 08 19 PM" src="https://github.com/user-attachments/assets/45ee78f0-4d2b-4da0-924a-a26dec6d754a">

## Made more orders
This is to confirm its working the right way
<img width="1497" alt="Screenshot 2024-08-12 at 7 08 43 PM" src="https://github.com/user-attachments/assets/49663d8e-7727-49e2-af2c-19b1c086774d">

<img width="1504" alt="Screenshot 2024-08-12 at 7 08 54 PM" src="https://github.com/user-attachments/assets/47f32ccd-814d-4a6a-a703-065e35731a01">



