# MIGRATE DATA INTO AWS RDS DATABASE USING FLYWAY


## In this project, I migrated our data from my local machine into AWS RDS database. I am willing to guide you in all the steps to get it done yourself


### **FIRST STEP**


#### Create a 3 Tier AWS VPC


- 1st Tier: 1st tier which is in public subnet, we will have bastion, ELB and NAT Gateway


- 2nd Tier we will have private subnet that will hold our websites is EC2 instances


- 3rd Tier - Another private subnets that will have our database and it be in different AZs for high availability and fault tolerance.


And then we will create internet gateway and route gateway to allow some resources in our VPC have access to the internet


### SOLUTIONS


- Create a VPC with 10.0.0.0/16 CIDR

- Give name and leave others as default

- Create

................................................................................

**Enable DNS hosting in VPC**


- click on the VPC to open

- At the top click on action button

- Select EDIT VPC settings

- Under DNS settings, enable DNS host name

- Save!

.................................................................................

**Create internet gateway for the VPC**


Go to internet gateway and create it. Only give name and Create.

.................................................................................

**Attach it to a VPC ...

Note: One VPC can be attached to IGW at a time

once the IGW is open, click on action and you see attach to a VPC, select your VPC and attach

Good!it will show attached

....................................................................................

**Create 2 public subnet in 2 AZs**

- From subnet section, click on create subenet

- Select the VPC created, give a name as public-subnet-AZ1

- Under Availability zone, select us-east-1a AZ.

- Give the CIDR as 10.0.0.0/24

- Create subnet



**Create another public subnet**


- Use same method but select another AZ ..east-us-1a with CIDR as 10.0.1.0/24

- Create

....................................................................................

**Enable auto IP assigned settings for the public subnets**


You are enabling this so that anytime you launch ECS instance in the subnet, IP address will be assigned automatically.


- Select subnet in AZ1 and from action button, click on edit subnet Ssetting.


- Tick the enable auto-assign IP address button and save

...............................................................................

**Create a public Route Table**

Before you start creating, you should see a route table which has been created already. It was created automatically when you created your VPC and it is called main Route table. it is private by default


**Let's create Route table**


- Go to routetable section and click on create route table

- Give it name as public-RT

- select the VPC and create RT

...................................................................................

**Add a public route to the public route table.**


- select the Route table created, from action click on edit route

- click on add, destination is 0.0.0.0/0 (don't use this if it is production) then target is IGW created. save it


....................................................................................


**Associate the 2 public subnets we created with the Route table


- From the Route table, go to subnet association and click on edit subnet association.

- select both 2 public subnets and associate


** Create 4 private subnets

- Click create subnet, select the VPC, name as Private-App-subnet-AZ1

- Use us-east-1a

- CIDR as 10.0.2.0/24

- Create it


..................................................................................

**create second private subnet**


- same method but use 10.0.3.0/24 as CIDR and Private-App-subnet-AZ2 in Uus-east-1b


..................................................................................

**create 3rd subnet - Data subnet**


- Give name as Private-Data-subnet-AZ1,  use 10.0.4.0/24 as CIDR in us-east-1a


....................................................................................

**create 4th subnet - Data subnet**


- same method but use 10.0.5.0/24 as CIDR and Private-Data-subnet-AZ2 in us-east-1b

**Done!**


**Difference between a Public subnet and Private subnet**


- When you create a route table and associate it Internet gateway and then link you subnet to it, it becomes public because it can be accessed from the internet.

- If you associate a subnet to the Route table that can't be accessed from the internet, then the subnet is private

- The 4 private subnets we created are private, no association with the pubic Route table explicitly. Hence, they are automatically associated with the main route table that was automatically created when you created the VPC
  The main RT is only routing traffic locally

...................................................................................

### Create a NAT Gateway In public subnet AZ1


- Go to NAT gateway section and click create 

- Give it a name as Nat-GW-AZ1

- Select the public subnet AZ1 

- Connectivity type should remain private

- under Elastic IP allocation ID, click on allocate Elastic IP and it will be allocated

- Elastic IP is also known as static IP address

- Click create!

....................................................................................

**Lets create a private Route table**


- Go to Route Table and click create

- Give name as Prvate-RT-AZ1

- Select your VPC

- Then Create


**Add a route to it**


- Under route, select edit route

- click add route

- use 0.0.0.0/0, under target, select NAT gateway

- Save!


**So now it will be going through the NAT gateway to the internet**

..............................................................................

**Associate the Private-App-subnet-AZ1 to this RT**


- We will associate it will Private-App-subnet-AZ1 and private-Data-subnet-AZ1

- Click subnet association, then click edit subnet association


Select Private-App-subnet-AZ1 and private-Data-subnet-AZ1 and associate

..............................................................................

**Create 2nd NAT gateway using the public-subnet-aZ2**


- Give name as NAT-GW-AZ2,

- Select Public-subnet-AZ2

- Click allocate elastic IP and create

...............................................................................

**Create another private route table**

- click on create RT

- Give name as Private-RT-AZ2
- Select VPC and create


- Add a route to the Private-RT-AZ2

- under the Private-RT-AZ2, click on route..edit route

- click add route..select destination as 0.0.0.0/0 and under target, sleect NAT-GW-AZ2

- Save

........................................................................................

**Associate the RT which is Private-RT-AZ2 with private-subnet-AZ2 & private-subnet-AZ2**


- click on subnet association

- edit subnet association

- select Private-App-AZ2 and Private-Data-subnet-AZ2

- Save it


**So from what we have done, the resources in the private subnets can have access to the internet via NAT gateway**


Note: Create NAT gateways only in public subnets


**To use Flyway to migrate data into an RDS instance in the private subnet, you need 2 security group**


1st SG: Bastion SG, we will attach it to the bastion host. on this SG, you open port 22 from your IP address

2nd SG: for RDS databse: we will attach it to RDS DB, you will open port 3306. Only allow traffic if it is coming from bastion host

...........................................................................

### Lets create SSH SECURITY GROUP

- Go to SG and click Create SG
- 
- Give name as Bastion-host-SG

- Same name for description 

- Select your VPC

- Click on add rule

- type is SSH
-
- source is My IP (limit it your IP address for security best practice)

- Create it

..........................................................................

### Create Database Security Group**

- Give name

- Same name for description 

- Select your VPC

- Click on add rule

- Type is Custom TCP, port is 3306, source should be bastion host SG you created previously. 

- Click create



By so doing you have limited the traffic to your database using bastion host SG


...........................................................................


**Create RDS Database in private subnet**

- Go to RDS service 

- 1st create subnet group. It will allow us specify which subnet we want to create the database in

- Go to subnet group section and click create subnet group

- Give it name as database-subnet

- Use same as decription

- Select your VPC

- Under add subnets, select AZ as us-east-1a and us-east-1b

- Under subnets according to our reference architecture, we have our private data subnet created with CIDR 10.0.4.0/24 in us-east1A..select it
and as well we have the second private data subnet using CIDR 10.0.5.0/24 in us-east-ib..select it

- Then create subnet group


................................................................

### Create Database


- Go to databases, click create database

- use standard create

- Under engine option, choose Mysql

- leave the default latest version as at the time of this documentation it is 8.0.28

- Under template, select Dev/test

- under AZ, select single DB instance

- Under settings

- give name, username and password


**PLEASE SAVE THIS PASSWORD SOMEWHERE!** ....continue with the steps

- Under instance config, use burstable classes

- Toggle previous generation classes so you use db.t2.micro

- Leave storage as default

- Under connectivity, leave the "Don’t connect to an EC2 compute resource" selected

- Select your VPC

- Under subnet group, ensure the one you created is selected

- Under public access, leave it as no

- Choose exiting security group...select the Database SG you created 

- Under Availability zone, select us-east-1b

- Under database authentication, leave it as password authentication

- Leave monitoring as default

- Under additional configuration, click the drop down arrow

- Under database option, give initial database name


NOTE DOWN YOUR INSTANCE IDENTIFIER NAME, DATABASE NAME, PASSWORD, USERNAME

You will need them later!


- Leave others as default and Create database

.....................................................................

### Create key pair

- Go to EC2 services and click on keypairs

- click on Create

- give name

- Type is RSA

- key format is Pem


Create and it will be downloaded to your PC


**Next Step**


- Open powershell in your PC

Note the directory your powershell is opening from..we have to move our keypair to that location so that when we run our ssh command later, our keypair will be in the same location we will ssh from


How to do it?


Go to the download folder or locate you rkeypairs in your PC, cut it and paste it in the exact folder your powershell opens from

Good job!


Note: When you create a keypair in AWS, 2 keys are created. The public and private keys

The one downloaded to your PC is private key


.....................................................................................

### Set up a bastion host we will use to SSH into the RDS instance in the private subnet


- Go to EC2 services, Click on launch instance

- Give it name as Bastion-host

- Use Amazon linux 2 ..free tier

- Instance type is t2 micro

- Under keypair...select the keypair created

- In network setting section, click edit

- select your VPC

- select public-subnet-AZ1

- Select existing security group and choose the bastion host SG

- Launch EC2 instance


............................................................................

### Downloading and Installing Flyway on Your Computer


Flyway is a database migration tool

- Go to the browser, type flyway

- click on the option that has flywaydb.org

- under the community section which is free, click on download 

- On the page, choose your OS.

- I will download windows OS

- click on your OS download link

and it will start downlaoding



- Once completed, go to your download folder and you will see it is in zip folder

- just right click and click extract all to extract it


**Next line is optional**

Once extracted, cut the folder and go the same location where your powershell opens from and paste it there


But you can choose to leave it in download folder

.............................................................................................

### Update the flyway configuration file

- In the flyway configuration file, you will enter the credentials of the database you want to connect to


- copy the configurqation file below

```
flyway.url=jdbc:mysql://localhost:3306/
flyway.user=
flyway.password=
flyway.locations=filesystem:sql
flyway.cleanDisabled=false
flyway.defaultSchema=<schema name>
```

**Next step**


 - Go your PC and open VS code..open folder to the location you saved the flyway folder we downlaoded previously, select and open in VS code

- Once open, select conf and and open flyway.conf file

- in this file, it shows some flyway options and some other deafult config file

- delete all and replace with the flyway file copied from github 

- Edit the file to connect to your RDS database

- go your RDS, select your database instance and on the configuration section, click on configuration

- copy out your database name

- Go back to vs code where you pasted the configuration file

-  In the file line, add the name...example
-  

**flyway.url=jdbc:mysql://localhost:3306/myappDB**

- Give the username of your database in the second line

- Type the DB password you saved in 3rd line

- Save it 

........................................................................................

### Organise SQL Scripts in Flyway

we must add our SQL script to the SQL directory within the flyway folder

The SQL script for the data we will migrate into the RDS database can be downloaded using the link below

https://drive.google.com/file/d/1H5Rdb15QsyaXF_Xauj9ODDP0O-yy3H5a/view


- Once downloaded, from file explorer, right click and cut the file

- Go to the flyware folder you downloaded earlier, click on it, open the flyway folder, click on sql folder and then paste the file

- You can delete the "put your sql migrations here" file there because you won't need it

- Go back to your VS code where we previously opened the flyway folder

- Go to SQL folder, click the downward arrow and you will see the rentzone-db.sql script file. Right click on the file click remain

- place your cursor at the beginning of name and type V1__ 

ie  V1 and then 2 underscores

so name will look like this

V1__rentzone-db.sql


**Why did we rename it? Because in order for flyway to migrate your data, you must name your script that way**

........................................................................................
 
### Securely Run Flyway Migrate with an SSH Tunel

This is to Setup SSH tunel from your local computer to the bastion host. Once done, then you can use Flyway to migrate your data to RDS database


Copy this command below


$ ssh -i <key_pier.pem> ec2-user@<public-ip> -L 3306:<rds-endpoint>:3306 -N

  
- I am using windows PC so this command works

- I will replace this command with my key and IP address of the bastion host and RDS endpoint. You should do same

So with this command, you can set up the SSH tunnel


**Note: Run this command in your powershell in same location your saved your keypair**


- Go to Visual studio code where your flyway folder is, at the top click on terminal and open new terminal

You will notice it opened into your flyway folder.

Run the command below
  
$ cd ..

  
- If you are in the user directory where you saved your keypair

- use **ls command** to ensure you can see the key
  
- Then run the command
  

- confirm with yes

  
**If the shell promt is hanging without progress then the bastion host tunnel is setup**



**Next step is to open another terminal**
  

- Go the left hand side where you have flyway file, right click on it and click open in integrated terminal

- New terminal will be opened
  

But remember the SSH terminal tunnel will still be there in the first terminal opened ..it will still be there runnung (hanging :)


So in the new terminal, run the command below


**$ .\flyway migrate**


Give it sometime and it will migrate your scripts successfully



![succeesful migration](https://user-images.githubusercontent.com/99098585/213942314-f2160641-dd57-410c-acc4-4233631be701.PNG)



  **CONGRATULATION!!** 🤗



### HOW TO CONFIRM THE TABLES OR DATA IN YOUR DATABASE



You can use the MySQL command-line client or a GUI client like MySQL Workbench to connect to your RDS instance and run SQL queries to view the data in the tables. For example, you can run the SELECT * FROM <table_name>; command to view all the data in a specific table.














 

