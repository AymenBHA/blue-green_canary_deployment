# blue-green_canary_deployment (AWS)
Blue-green and canary
In modern cloud environments, deploying new versions of an application shouldn't mean taking your system offline. This project builds a highly resilient infrastructure by deploying two identical, fully isolated environments (Blue and Green) behind an AWS Application Load Balancer (ALB).

we insure transition traffic seamlessly from your live environment (Version 1 / Blue) to our updated environment (Version 2 / Green) using a Canary deployment strategy. Rather than flipping a switch and hoping for the best, a Canary strategy allows us to route a small percentage of user traffic (e.g., 20%) to the new version to monitor for errors before committing to a full 100% cutover. This ensures zero downtime and minimizes the blast radius if an update fails.

=========================================================================================================
We need to create the foundational network to host our servers. An Application Load Balancer requires at least two public subnets in different Availability Zones.

Tasks:

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



======================================================================================================

We will launch our first server, representing the current "live" version of our application.

Tasks:

Step 1: Launch Blue Instance

Dashboard: EC2 > Instances > Launch instances
Name: Blue-Server
AMI: Amazon Linux 2023
Instance Type: t3.micro
Key Pair: Proceed without a key pair (not needed for this lab).
Network Settings: Click Edit.
VPC: BG-VPC
Subnet: Public-1
Security Group: Create security group > Name: Web-SG > Add rule: Allow HTTP (80) from Anywhere (0.0.0.0/0).

Advanced Details (User Data): Scroll to the bottom and paste the blue-ec2 file content



Click: Launch instance



======================================================================================================
We will now launch our updated server, representing the new version of our application we want to test.

Tasks:

Step 1: Launch Green Instance

Dashboard: EC2 > Instances > Launch instances
Name: Green-Server
AMI: Amazon Linux 2023
Instance Type: t3.micro
Key Pair: Proceed without a key pair
Network Settings: Click Edit.
VPC: BG-VPC
Subnet: Public-2
Security Group: Choose Select existing security group > Choose Web-SG. (Note: Uncheck the default VPC security group if it is selected).
Advanced Details (User Data): Scroll to the bottom and paste the green-ec2 file content



Click: Launch instance

======================================================================================================
Target groups tell our Load Balancer where to send traffic. We need one for each environment.

Tasks:

Step 1: Create Target Group (Blue)

Dashboard: EC2 > Target Groups > Create target group
Type: Instances
Name: TG-Blue
VPC: BG-VPC
Protocol: HTTP (80) > Click Next.
Register Targets: Select Blue-Server > Click Include as pending below.
Click: Create target group
Step 2: Create Target Group (Green)

Repeat the exact steps above, but name it TG-Green and register the Green-Server.

Note:
Are both target groups created?
Does TG-Blue contain the Blue-Server and TG-Green contain the Green-Server?

======================================================================================================

The Load Balancer will act as the single entry point for our users, allowing us to seamlessly shift traffic between the backend target groups.

Tasks:

Step 1: Create Load Balancer

Dashboard: EC2 > Load Balancers > Create load balancer
Type: Application Load Balancer (ALB) > Click Create.
Name: BlueGreen-ALB
Network mapping: Select BG-VPC | Check both us-east-1a (Public-1) and us-east-1b (Public-2).
Security Groups: Deselect the default, and select Web-SG.
Listeners and routing: Protocol HTTP (80) -> Default action: Forward to TG-Blue.
Click: Create load balancer
Note: Wait a few minutes for the Load Balancer status to become Active before proceeding.



======================================================================================================

Currently, 100% of traffic goes to the Blue instance. Let's shift it using a Canary deployment strategy.

Tasks:

Step 1: Verify Status Quo

Copy the DNS Name of your ALB and paste it into a browser tab (ensure it starts with http://).
Verify: You should see 🟦 VERSION 1 (BLUE).
Refresh multiple times. It should stay Blue.
Step 2: Modify Listener Rule (Canary Deployment)

Go to your Load Balancer > Listeners and rules tab.
Select the Listener (HTTP:80) > Click Manage rules > Edit rule (Pencil icon).
Under Routing Actions, change the weights:
Target Group 1: TG-Blue | Weight: 80
Target Group 2: TG-Green | Weight: 20
Click: Save changes
Step 3: WAIT: The Propagation Pause

Wait 30 to 60 seconds for the rules to update.
Step 4: Test the Shift

Refresh your browser repeatedly.
Result: Roughly 80% of the time you see BLUE, and 20% of the time you see GREEN.
Step 5: Full Cutover

Edit the Rule again.
Weights: TG-Blue: 0, TG-Green: 100.
Click: Save changes.
Result: 100% of users now see 🟩 VERSION 2 (GREEN).

======================================================================================================

Congratulations,

we have successfully executed an enterprise-grade deployment strategy in AWS!

By utilizing a Blue/Green architecture combined with a Canary traffic shift, you have learned how to safely introduce new application versions to a subset of users. This methodology minimizes downtime, drastically reduces risk, and ensures a seamless experience for your end-users.

Here is a recap of the skills we just mastered:

Advanced VPC Networking: Provisioning multi-AZ subnets to ensure high availability and fault tolerance for load balancers.
EC2 Provisioning: Automating web server configuration using User Data scripts upon boot.
Target Group Decoupling: Isolating distinct application environments into separate backend target groups for granular traffic control.
Application Load Balancers (ALB): Configuring listener rules to act as the central routing mechanism for your users.
Canary Deployments: Shifting precise percentages of live traffic between active environments to safely test new releases in production.

