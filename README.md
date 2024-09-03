# AWS_Basic_EC2_VP

Step-by-step guide to creating a virtual cloud network on AWS that includes a Linux webserver with both a public and private IPv4 address and a database server with only a private IPv4 address i.e. a cloud network with a webserver that can publicly access the internet and a database server that is isolated within the private subnet but accessible from the webserver.

### **Step 1: Set Up Your AWS Account**
1. **Sign in to AWS Management Console**: If you don’t have an AWS account, sign up at [aws.amazon.com](https://aws.amazon.com/).
2. **Navigate to the AWS Management Console**: Once logged in, you will be directed to the AWS Management Console.

### **Step 2: Create a Virtual Private Cloud (VPC)**
1. **Navigate to the VPC Dashboard**:
   - In the AWS Management Console, type "VPC" in the search bar and select it from the list.
2. **Create a New VPC**:
   - Click on “Your VPCs” from the left sidebar, then click the "Create VPC" button.
   - **Name Tag**: Enter a name for your VPC (e.g., `MyVPC`).
   - **IPv4 CIDR Block**: Enter the IP range (e.g., `10.0.0.0/16`).
   - **Tenancy**: Select “default” unless you have specific needs for dedicated instances.
   - Click “Create VPC.”

### **Step 3: Create Subnets**
1. **Create a Public Subnet**:
   - Go to "Subnets" on the left sidebar, then click “Create Subnet.”
   - **Name Tag**: Enter a name for your public subnet (e.g., `PublicSubnet`).
   - **VPC**: Select the VPC you created earlier.
   - **Availability Zone**: Choose an availability zone (e.g., `us-east-1a`).
   - **IPv4 CIDR Block**: Enter a subnet range within your VPC (e.g., `10.0.1.0/24`).
   - Click “Create Subnet.”
2. **Enable Auto-assign Public IP for Public Subnet**:
   - Select the public subnet you just created.
   - Click on the “Actions” dropdown and select “Modify auto-assign IP settings.”
   - Check “Enable auto-assign public IPv4 address” and save changes.
3. **Create a Private Subnet**:
   - Repeat the steps to create another subnet, but name this one `PrivateSubnet`.
   - **IPv4 CIDR Block**: Enter a different range within your VPC (e.g., `10.0.2.0/24`).
   - Do not enable auto-assign public IP for this subnet.

### **Step 4: Create an Internet Gateway**
1. **Create the Internet Gateway**:
   - Go to "Internet Gateways" on the left sidebar, then click “Create Internet Gateway.”
   - **Name Tag**: Enter a name (e.g., `MyInternetGateway`).
   - Click “Create Internet Gateway.”
2. **Attach the Internet Gateway to Your VPC**:
   - Select the newly created Internet Gateway.
   - Click “Actions” and then “Attach to VPC.”
   - Select your VPC and click “Attach Internet Gateway.”

### **Step 5: Set Up Route Tables**
1. **Create a Route Table for Public Subnet**:
   - Go to "Route Tables" on the left sidebar, then click “Create route table.”
   - **Name Tag**: Enter a name (e.g., `PublicRouteTable`).
   - **VPC**: Select your VPC.
   - Click “Create route table.”
2. **Edit Routes for Public Subnet**:
   - Select the public route table you just created.
   - Under "Routes" tab, click “Edit routes” and then “Add route.”
   - **Destination**: Enter `0.0.0.0/0`.
   - **Target**: Select the Internet Gateway you created earlier.
   - Click “Save changes.”
3. **Associate the Route Table with the Public Subnet**:
   - Go to the “Subnet associations” tab and click “Edit subnet associations.”
   - Select the public subnet and click “Save associations.”
4. **Create and Associate Route Table for Private Subnet**:
   - Repeat the steps to create another route table for the private subnet (e.g., `PrivateRouteTable`), but do not add any routes to the Internet Gateway.
   - Associate this route table with your private subnet.

### **Step 6: Launch EC2 Instances**
1. **Launch a Linux EC2 Instance for the Webserver**:
   - Navigate to the "EC2 Dashboard."
   - Click “Launch Instance.”
   - **Name and OS**: Enter a name (e.g., `WebServer`), and choose an Amazon Machine Image (AMI) such as Amazon Linux 2.
   - **Instance Type**: Choose the instance type (e.g., `t2.micro`).
   - **Network Settings**: 
     - **VPC**: Select your VPC.
     - **Subnet**: Choose the public subnet.
     - **Auto-assign Public IP**: Ensure it’s set to enable.
   - **Configure Storage**: Use default settings or customize based on your needs.
   - **Security Group Settings**: 
     - Create a new security group (e.g., `WebServerSG`).
     - Add rules for HTTP (`port 80`), HTTPS (`port 443`), and SSH (`port 22`).
   - Review and launch the instance.
2. **Launch a Linux EC2 Instance for the Database Server**:
   - Follow the same steps to launch another EC2 instance, but with these changes:
     - **Name**: Enter `DatabaseServer`.
     - **Subnet**: Choose the private subnet.
     - **Auto-assign Public IP**: Disable this option.
   - **Security Group Settings**:
     - Create a new security group (e.g., `DatabaseServerSG`).
     - Add a rule to allow MySQL/Aurora (`port 3306`).
     - Add a custom rule to allow traffic from the WebServer (add the WebServerSG security group as a source).
   - Review and launch the instance.

### **Step 7: Configure the Webserver and Database Server**
1. **Access the Webserver via SSH**:
   - Use SSH to connect to the Webserver instance using its public IP address.
   - Example command: 
     ```bash
     ssh -i "your-key-pair.pem" ec2-user@your-webserver-public-ip
     ```
2. **Install a Web Server**:
   - On the Webserver, install a web server like Apache.
     ```bash
     sudo yum update -y
     sudo yum install httpd -y
     sudo systemctl start httpd
     sudo systemctl enable httpd
     ```
3. **Access the Database Server via Private IP**:
   - From the Webserver, connect to the Database Server using its private IP.
   - Example command to install and connect to MySQL:
     ```bash
     sudo yum install mysql -y
     mysql -h your-database-private-ip -u your-username -p
     ```

### **Step 8: Final Testing and Verification**
1. **Verify Webserver Accessibility**:
   - Access the web server via its public IP address in a web browser.
2. **Verify Database Connectivity**:
   - From the Webserver, ensure you can connect to the database server via its private IP.
3. **Configure Webserver to Use the Database**:
   - If you had installed something like WordPress, edit your web application’s configuration to point to the database server’s private IP address, otherwise you can skip this.

### **Step 9: (Optional) Set Up Additional Security Measures**
1. **Set Up NACLs and Additional Security Groups**:
   - Consider setting up Network Access Control Lists (NACLs) for an extra layer of security.
2. **Enable Logging and Monitoring**:
   - Enable CloudWatch monitoring for your EC2 instances and VPC.

