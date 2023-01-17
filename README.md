# MIGRATE DATA INTO RDS DATABASE USING FLYWAY

**FIRST STEP**


- Create a 3 Tier AWS VPC


a. 1st tier: 1st tier which is in public subnet, we will have bastion, ELB and NAT gateay


b. 2nd Tier we will have private subnet that will hold our websites is EC2 instances


c. 3rd tier - Another private subnets that will have our database


and it be in diff AZs for high availability and fault tolerance


And then we will create internet gateway and route gateway to allow some resources in our VPC have access to the internet


**LETS CREATE!**

Create a VPC with 10.0.0.0/16b CIDR

give name leave others as default...create


**Enable DNS hosting in VPC**

click on the VPC to open, then at the top click on action, select EDIT VPC settings. Under DNS settings, enable DNS host name
Save!


**Create internet gateway for the VPC**


Go to internet gateway and create it. Only give name and Create.


**Attach it to a VPC ...

Note: One VPC can be attached to IGW at a time

once the IGW is open, click on action and you see attach to a VPC, select your VPC and attach

Good!it will show attached


**Create 2 public subnet in 2 AZs**

From subnet section, click on create subenet
Select the VPC created, give a name as public-subnet-AZ1
Under Availability zone, select us-east-1a AZ.

Give the CIDR as 10.0.0.0/24
Create subnet

Create another public subnet

Same method but select another AZ ..east-us-1a with CIDR as 10.0.1.0/24

Create


**Enable auto IP assigned settings for the public subnets**


So anytime you launch ECS instance in the subnet, IP address will be assigned automatically.


Select subnet in AZ1 and from action button, click on edit subnet Ssetting.


tick the enable auto-assign IP address button and save


**Create a public Route Table**

Before you start creating, you should see a route table which has been created already. It was created automatically when you created your VPC and it is called main Route table
it is private by default


**Let create Route table**


- Go to routetable section and click on create route table

- Give it name as public-RT

- select the VPC and create RT



**Add a public route to the public route table.**


- select the Route table created, from action click on edit route

- click on add, destination is 0.0.0.0/0 (don't use this if it is production) then target is IGW created. save it


**Associate the 2 public subnets we created with the Route table


From the route table, go to subnet association and click on edit subnet association.

select both 2 public subnets and associate


** Create 4 private subnets

- Click create subnet, select the VPC, name as Private-App-subnet-AZ1

- Use us-east-1a

- CIDR as 10.0.2.0/24


Create it


**create second private subnet**


- same method but use 10.0.3.0/24 as CIDR and Private-App-subnet-AZ2 in Uus-east-1b


**create 3rd subnet - Data subnet**


Give name as Private-Data-subnet-AZ1,  use 10.0.4.0/24 as CIDR in us-east-1a


**create 4th subnet - Data subnet**


same method but use 10.0.5.0/24 as CIDR and Private-Data-subnet-AZ2 in us-east-1b

Done!


**Difference between a Public subnet and Private subnet**


- When you create a route table and associate it Internet gateway and then link you subnet to it, it becomes public because it can be accessed from the internet.

- If you associate a subnet to the Route table that can't be accessed from the internet, then the subnet is private

- The 4 private subnets we created are private, no association with the pubic Route table explicitly. Hence, they are automatically associated with the main route table that was automatically created when you created the VPC
  The main RT is only routing traffic locally



**Create a NAT Gateway In public subnet AZ1**


- Go to NAT gateway section and click create 

- Give it a name as Nat-GW-AZ1

- Select the public subnet AZ1 

- Connectivity type should remain private

- under Elastic IP allocation ID, click on allocate Elastic IP and it will be allocated

- Elastic IP is also known as static IP address

- Click create!


**Lets create a private Route table**


- Go to Route Table and click create

- Give name as Prvate-RT-AZ1

- Select your VPC

- Then Create


**Add a route to it**

