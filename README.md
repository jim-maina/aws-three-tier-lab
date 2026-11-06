AWS Three-Tier Web App Lab
A hands-on project where I built the foundation of a three-tier web architecture on AWS by hand, then tested it and tore it down. This is the first stage of my cloud architecture portfolio. Later stages rebuild the same design as Terraform code, add a CI/CD pipeline, and harden it.
I'm new to AWS, so this README also records what I learned at each step.
Architecture
                    Internet
                       |
                       v
          +---------------------------+
          |  Application Load Balancer |   public subnets (2 zones)
          |  cloud-lab-alb  (HTTP 80)  |
          +---------------------------+
                       |
                       v
          +---------------------------+
          |  EC2 web server (nginx)    |   public subnet (eu-north-1a)
          |  cloud-lab-web             |
          +---------------------------+
                       |
                       v
          +---------------------------+
          |  RDS MySQL database        |   private subnets (2 zones)
          |  cloud-lab-db              |   no public access
          +---------------------------+

All inside one VPC: cloud-lab-vpc (10.0.0.0/16), region eu-north-1 (Stockholm)


What I built
Piece	What it is	Why it's there
VPCcloud-lab-vpc	My own private network	Isolates the project from everything else
4 subnets	2 public, 2 private, across 2 availability zones	Public ones face the internet, private ones hide the database
Internet gateway	The door to the internet	Lets public subnets reach the internet
EC2 instancecloud-lab-web	A small server (t3.micro, Amazon Linux) running nginx	Serves the website
Application Load Balancercloud-lab-alb	Sits in front of the server	Single entry point, can spread traffic over many servers later
Target groupcloud-lab-tg	List of servers the load balancer sends traffic to	Lets the load balancer check server health
RDS MySQLcloud-lab-db	Managed database (db.t4g.micro, 20 GiB)	Stores data, kept in the private tier
Security groups	Firewalls for the load balancer, web server and database	Control who can talk to what

Security decisions
·	Chain of trust between firewalls. The load balancer accepts HTTP from the internet. The web server accepts HTTP only from the load balancer's security group. The database accepts MySQL (port 3306) only from the web server's security group.
·	Proof it works. After locking down the web server, its own public IP stopped responding, while the load balancer address still worked. The load balancer is the only way in.
·	Database in private subnets. I created a dedicated DB subnet group containing only the private subnets and set Public access: No. The default "create new" subnet group would have included the public subnets too.
·	No SSH rule. I didn't open SSH to the internet at all.
·	Account security. I turned on two-factor authentication on the login that controls my AWS account, and set a zero-spend budget alert.
Cost decisions
·	Used the AWS Free plan (credit-based) and the smallest instance sizes.
·	Chose no NAT gateway because it charges by the hour.
·	Used a single-AZ database to keep costs down. For a production shop I would enable Multi-AZ.
·	Deleted the database, load balancer and server when I finished so they stop using credits.

Screenshots
	

	1. VPC resource map. The VPC with its public and private subnets in two zones.

	2. EC2 instance. The web server running in the public subnet.

	3. Target group health. The load balancer sees the server as healthy.

	4. Load balancer. The Application Load Balancer is active.

	5. The site through the load balancer. The page loads via the load balancer address.

	6. RDS database. Not publicly accessible, in the private subnet group, with its own security group.

What I learned
·	A VPC is a private network, and subnets split it into public and private areas.
·	A security group is a firewall, and one security group can be the allowed source for another. This is better than opening things to 0.0.0.0/0.
·	A load balancer needs at least two public subnets in different availability zones, and a target group to know where to send traffic.
·	A "timed out" error right after launch usually means the resource is still starting, not that something is broken.
·	Console settings can quietly change the design (for example the default DB subnet group), so it's worth checking each section before creating anything.
·	Least privilege: only allow the traffic that is actually needed.
Limitations (honest notes)
·	The web server is not yet connected to the database. The firewall rules allow it, but I haven't set up the connection.
·	The site uses plain HTTP only. A real shop needs a domain name and an HTTPS certificate.
·	The web server sits in a public subnet. A stricter design would place it in a private subnet behind the load balancer.
·	This was built by hand in the console. It isn't reproducible yet, which is the point of the next stage.
Next steps
1.	Rebuild the whole stack as Terraform code (variables and modules for network, compute and database).
2.	Add a GitHub Actions pipeline that checks and applies the Terraform.
3.	Harden it: tighter IAM permissions, logging with CloudTrail and VPC flow logs, a cost alert.
4.	Test it from the attacker's side in a throwaway lab (for example with Kali tools such as nmap), fix what I find, and document the before and after.
5.	Study for the AWS Solutions Architect Associate exam.

Built by [James Maina], November 2026.
