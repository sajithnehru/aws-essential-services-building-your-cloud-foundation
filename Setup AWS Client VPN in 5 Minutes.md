## Setup AWS Client VPN in 5 Minutes
**Scenario**  
We have an EC2 instance in a private subnet without a public IP address. We want to connect to this instance remotely, install an Apache or Nginx web server, and be able to ping the instance from our client machine.

**Prerequisites**  
- A VPN client (OpenVPN/AWS VPN client) on your client machine.  
- Connection from a Linux terminal (Git Bash /Ubuntu on Windows).  
- AWS Console access with the privilege to import certificates to AWS Certificate Manager.
- The EC2 instance is in a private subnet.

If you need assistance setting up an EC2 instance, please refer to Step 0, else start with Step 1.  

**Step 0:**   
Setting Up a New VPC and EC2 Instance with Apache Web Server
Create a New VPC:
- Go to the VPC service.
- Click Create VPC.
- Select VPC and More.
- Check Autogenerate.
- Enter a VPC name prefix (e.g., My-VPC01).
- Set the IPv4 CIDR block to 10.10.10.0/16.
- Leave the remaining settings as default and click Create VPC.
- Launch an EC2 Instance:  

Go to the EC2 service.
- Click Launch Instance.
- Enter a name for the instance (e.g., webserver-01).
- Select Amazon Linux 2023 AMI.
- Choose an existing key pair or create a new one.
- Under Network Settings, click Edit.
- Select your new VPC in the VPC dropdown.
- Choose your private subnet in the Subnet dropdown.
- Create a new security group if needed.
- Expand Advanced details and scroll to the bottom.
- In the User data section, paste the script provided at the end of this documentation.
- Click Launch Instance.

Note: Record the IP address of the instance, as it will be used in Step 9.

**Steps to Configure AWS Client VPN**  
**1. Set Up Certificates**  
Open a Linux terminal (Git  terminal used here) and run the following commands:  

Clone the OpenVPN easy-rsa repository:  

```
git clone https://github.com/OpenVPN/easy-rsa.git
cd easy-rsa/easyrsa3
```

Initialize the PKI environment:  


```
./easyrsa init-pki
```
Create a CA certificate:  


```
./easyrsa build-ca nopass
```

Create a server key and certificate:  


```
./easyrsa build-server-full server nopass

```
Create a client key and certificate:  


```
./easyrsa build-client-full client1.domain.tld nopass
```
Copy the certificates and keys to a designated folder:  

```
mkdir ~/certificate-folder/
cp pki/ca.crt ~/certificate-folder/
cp pki/issued/server.crt ~/certificate-folder/
cp pki/private/server.key ~/certificate-folder/
cp pki/issued/client1.domain.tld.crt ~/certificate-folder/
cp pki/private/client1.domain.tld.key ~/certificate-folder/  
```

**2. Import Certificates into AWS Certificate Manager** 

- Go to the AWS Console > Certificate Manager > Import a Certificate.
- Insert the certificate, key, and CA one by one:
- Server certificate: Tag Name: Server
- Client certificate: Tag Name: Client1.domain

**3. Create a Client VPN Endpoint**  
- Navigate to VPC > Virtual Private Network (VPN) > Client VPN Endpoints > Create Client VPN Endpoint.

**Configure the endpoint:**  

- Client IPv4 CIDR: Any CIDR value (e.g., 20.0.0.0/16).
- Authentication Information: Select the server certificate.
- Use mutual authentication: Select the client certificate.
- Enable split tunnel: To ensure your internet connection remains active.
- Select your VPC and Security Group.

- Click on Create Client VPN Endpoint. It may take 3 to 10 minutes to be ready.

**4. Associate Target Network**  
- Open the endpoint and go to Target Network Association.
- Associate the target network with your VPC and subnet.

**5. Configure Security Group**  
- Add required rules to allow connections to the instance.

**6. Add Authorization Rule**  
- Navigate to Authorization Rules.
- Add an authorization rule:
- Destination Network: Your VPC CIDR.
- Allow access to all users: Yes.

**7. Download and Configure Client Configuration File**  
Click on Client Configuration at the top right corner and save the file to a secure location on your client machine.

Open the configuration file in a text editor.

Add the client key and certificate:

```
<key>
-----BEGIN PRIVATE KEY-----
MIIEvAIBADANBgkqh...
LnBsK5NIZczmD6sJ0oKj6g==
-----END PRIVATE KEY-----
</key>
<cert>
-----BEGIN CERTIFICATE-----
MIIDYTCCAkmgAwIBAgIRANsJ3uIm...
aqACtis=
-----END CERTIFICATE-----
</cert>
```
8. Connect Using Client VPN Tool
Open the Client VPN tool and import the configuration file.
Click on Connect.

*** 9. Validate Connectivity ***   
- Ping the private IP address of the instance.
- Load the instance's web server in a browser.
- User Data Script for EC2 Instance
- Use the following script to install Apache, enable SSM, and allow ping responses:


```
#!/bin/

# Update packages
sudo yum update -y

# Install Apache web server
sudo yum install httpd -y

# Enable and start Apache
sudo systemctl enable httpd
sudo systemctl start httpd

# Install and configure SSM Agent
sudo amazon-linux-extras install amazon-ssm-agent -y
sudo systemctl enable amazon-ssm-agent
sudo systemctl start amazon-ssm-agent

# Allow HTTP traffic on port 80 (for Apache) and ICMP (for ping)
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-service=icmp
sudo firewall-cmd --reload

# (Optional) Verify firewall configuration
sudo firewall-cmd --list-all

#Test Apache with a basic index.html file
sudo echo "It works!" > /var/www/html/index.html  
```

References  
For more detailed information, refer to the AWS Client VPN documentation.  
https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/mutual.html  
https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/cvpn-getting-started.html  
