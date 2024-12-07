
## **![architecture](https://github.com/user-attachments/assets/af0733a1-8469-464b-a4c2-345fa371e79e)
Overview**
This project involves creating a secure and scalable Virtual Private Cloud (VPC) architecture with public and private subnets in two Availability Zones. It includes deploying a web server in one of the public subnets, and enabling private subnets to access the internet using a NAT Gateway.

---

## **Step 1: Set Up Your Environment**
1. **Sign in to AWS Console**: Log in to your AWS Management Console.
2. **Region Selection**: Choose the appropriate AWS region (e.g., `us-east-1`) where you want to deploy this architecture.

---

## **Step 2: Create a VPC**
1. Navigate to **VPC** → **Your VPCs** → **Create VPC**.
2. Configure the VPC:
   - **Name tag**: `lab-vpc`
   - **IPv4 CIDR Block**: `10.0.0.0/16`
   - **Tenancy**: Default
3. Click **Create VPC**.

---

## **Step 3: Create Subnets**
You will create four subnets (two public, two private) distributed across two Availability Zones.

1. **Create Public Subnet 1**:
   - Name: `lab-subnet-public1`
   - VPC: Select `lab-vpc`
   - Availability Zone: Choose `us-east-1a` (or any AZ in your region)
   - CIDR block: `10.0.0.0/24`
   - Enable: Auto-assign public IPv4 addresses.
   - Click **Create Subnet**.

2. **Create Private Subnet 1**:
   - Name: `lab-subnet-private1`
   - VPC: Select `lab-vpc`
   - Availability Zone: Choose `us-east-1a`
   - CIDR block: `10.0.1.0/24`
   - Leave **Auto-assign public IPv4 addresses** unchecked.
   - Click **Create Subnet**.

3. **Create Public Subnet 2**:
   - Name: `lab-subnet-public2`
   - VPC: Select `lab-vpc`
   - Availability Zone: Choose `us-east-1b` (or another AZ in your region)
   - CIDR block: `10.0.2.0/24`
   - Enable: Auto-assign public IPv4 addresses.
   - Click **Create Subnet**.

4. **Create Private Subnet 2**:
   - Name: `lab-subnet-private2`
   - VPC: Select `lab-vpc`
   - Availability Zone: Choose `us-east-1b`
   - CIDR block: `10.0.3.0/24`
   - Leave **Auto-assign public IPv4 addresses** unchecked.
   - Click **Create Subnet**.

---

## **Step 4: Create an Internet Gateway**
1. Navigate to **VPC** → **Internet Gateways** → **Create Internet Gateway**.
   - Name: `lab-igw`.
2. Attach the Internet Gateway to `lab-vpc`:
   - Select the Internet Gateway → Actions → Attach to VPC → Select `lab-vpc`.

---

## **Step 5: Configure Route Tables**
You need two route tables: one for public subnets and one for private subnets.

### **Create and Configure the Public Route Table**
1. Navigate to **VPC** → **Route Tables** → **Create Route Table**.
   - Name: `Public-Route-Table`
   - VPC: Select `lab-vpc`.
   - Click **Create**.
2. Add a route for internet access:
   - Select the `Public-Route-Table`.
   - Go to **Routes** → **Edit Routes** → **Add Route**:
     - Destination: `0.0.0.0/0`
     - Target: Select the **Internet Gateway** (`lab-igw`).
   - Save changes.
3. Associate the public subnets:
   - Go to **Subnet Associations** → **Edit Subnet Associations**.
   - Select `lab-subnet-public1` and `lab-subnet-public2`.
   - Save.

### **Create and Configure the Private Route Table**
1. Navigate to **VPC** → **Route Tables** → **Create Route Table**.
   - Name: `Private-Route-Table`
   - VPC: Select `lab-vpc`.
   - Click **Create**.
2. Add a route for internet access via NAT:
   - Select the `Private-Route-Table`.
   - Go to **Routes** → **Edit Routes** → **Add Route**:
     - Destination: `0.0.0.0/0`
     - Target: NAT Gateway (to be created in Step 6).
   - Save changes.
3. Associate the private subnets:
   - Go to **Subnet Associations** → **Edit Subnet Associations**.
   - Select `lab-subnet-private1` and `lab-subnet-private2`.
   - Save.

---

## **Step 6: Create a NAT Gateway**
1. Navigate to **VPC** → **NAT Gateways** → **Create NAT Gateway**.
   - Name: `lab-nat-gateway`.
   - Subnet: Select `lab-subnet-public1`.
   - Elastic IP: Allocate a new Elastic IP address.
2. Click **Create NAT Gateway**.
3. Go back to the `Private-Route-Table` (from Step 5) and update the route:
   - Target: Select the newly created NAT Gateway.

---

## **Step 7: Create Security Groups**
### **Web Server Security Group**
1. Navigate to **VPC** → **Security Groups** → **Create Security Group**.
   - Name: `Web-Server-SG`.
   - VPC: Select `lab-vpc`.

2. Configure **Inbound Rules**:
   - Type: HTTP, Port Range: 80, Source: `0.0.0.0/0`.
   - Type: SSH, Port Range: 22, Source: `Your IP Address`.

3. Configure **Outbound Rules**:
   - Destination: Allow all traffic (`0.0.0.0/0`).

---

## **Step 8: Launch a Web Server**
1. Navigate to **EC2** → **Instances** → **Launch Instance**.
   - AMI: **Amazon Linux 2**.
   - Instance Type: **t2.micro** (free tier eligible).
   - Network: Select `lab-vpc`.
   - Subnet: Select `lab-subnet-public2`.
   - Security Group: Select `Web-Server-SG`.

2. Complete the instance launch and connect to it:
   - Use SSH from your terminal:
     ```bash
     ssh -i <your-key.pem> ec2-user@<Public-IP>
     ```

3. Install and configure a web server (Apache):
   ```bash
   sudo yum update -y
   sudo yum install -y httpd
   sudo systemctl start httpd
   sudo systemctl enable httpd
   echo "Hello, World from Web Server!" | sudo tee /var/www/html/index.html
   ```

4. Verify the web server:
   - Open a browser and navigate to the instance's public IP. You should see **"Hello, World from Web Server!"**.

---

## **Step 9: Testing Private Subnet Connectivity**
1. Launch another EC2 instance in `lab-subnet-private2` without a public IP.
2. Ensure it can reach the internet via the NAT Gateway:
   - SSH into the instance and ping a public server (e.g., `ping google.com`).

---

## **Step 10: Clean Up Resources (Optional)**
If this setup is for practice, ensure you delete all resources to avoid incurring charges:
1. Terminate EC2 instances.
2. Delete the NAT Gateway, Internet Gateway, subnets, route tables, and VPC.

---

## **Conclusion**
This step-by-step guide outlines how to create a highly available and secure VPC architecture with public and private subnets. It also demonstrates how to deploy and test a web server while ensuring private subnet connectivity via a NAT Gateway.
