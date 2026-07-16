# blue-green_canary_deployment (AWS)
Blue-green and canary
In modern cloud environments, deploying new versions of an application shouldn't mean taking your system offline. This project builds a highly resilient infrastructure by deploying two identical, fully isolated environments (Blue and Green) behind an AWS Application Load Balancer (ALB).

we insure transition traffic seamlessly from your live environment (Version 1 / Blue) to our updated environment (Version 2 / Green) using a Canary deployment strategy. Rather than flipping a switch and hoping for the best, a Canary strategy allows us to route a small percentage of user traffic (e.g., 20%) to the new version to monitor for errors before committing to a full 100% cutover. This ensures zero downtime and minimizes the blast radius if an update fails.

We need to create the foundational network to host our servers. An Application Load Balancer requires at least two public subnets in different Availability Zones.



Step 1: Create VPC

Dashboard: VPC > Create VPC
Name: BG-VPC
IPv4 CIDR: 10.0.0.0/16
Click: Create VPC
Step 2: Create 2 Public Subnets

Dashboard: Subnets > Create subnet
Select VPC: BG-VPC
Subnet 1: Name Public-1 | AZ us-east-1a | CIDR 10.0.1.0/24
Subnet 2: Name Public-2 | AZ us-east-1b | CIDR 10.0.2.0/24
Click: Create subnet
Step 3: Enable Public IPs (Crucial)

Dashboard: Subnets
Select: Public-1
Actions: Edit subnet settings
Check: Enable auto-assign public IPv4 address > Save
Repeat this step for Public-2.
Step 4: Internet Gateway & Routes

IGW: Dashboard: Internet Gateways > Create BG-IGW > Actions: Attach to VPC > Select BG-VPC.
Route Table: Dashboard: Route Tables > Create BG-RT (Select BG-VPC) > Create.
Add Route: Select BG-RT > Routes tab > Edit routes > Add Route 0.0.0.0/0 → Target Internet Gateway (BG-IGW) > Save changes.
Associations: Select BG-RT > Subnet associations tab > Edit subnet associations > Select both Public-1 and Public-2 > Save associations.

note:
insure both subnets created in different Availability Zones
Did need to remember to enable auto-assign public IPs for both subnets
