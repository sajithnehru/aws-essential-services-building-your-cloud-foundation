## Setup AWS Client VPN in 10 Minutes
**Scenario**  
We have an EC2 instance in a private subnet without a public IP address. We want to connect to this instance remotely via SSH. 

**Prerequisites**  
- A VPN client (OpenVPN/AWS VPN client) on your client machine.  
- Connection from a Linux terminal (Git Bash /Ubuntu on Windows).  
- AWS Console access with the privilege to import certificates to AWS Certificate Manager.
- The EC2 instance is in a private subnet.

If you need assistance setting up an EC2 instance, please refer to Step 0, else start with Step 1.  
**Step 0:** Setting Up a New VPC and EC2 Instance  
Create a New VPC:
- Go to the VPC service.
- Click Create VPC.
- Select VPC and More.
- Check Autogenerate.
- Enter a VPC name prefix (e.g.,My-VPC01).
- Set the IPv4 CIDR block to 10.0.0.0/16.
- Leave the remaining settings as default and click Create VPC.
  
**Launch an EC2 Instance:**  
Go to the EC2 service.
- Click Launch Instance.
- Enter a name for the instance (e.g., webserver-01).
- Select Amazon Linux 2023 AMI.
- Choose an existing key pair or create a new one.
- Under Network Settings, click Edit.
- Select your new VPC in the VPC dropdown.
- Choose your private subnet in the Subnet dropdown.
- Create a new security group if needed.
- Click Launch Instance.

**Steps to Configure AWS Client VPN**  
**1. Set Up Certificates**  
Open a Linux terminal (Git  terminal used here) and run the following commands:  

a. Clone the OpenVPN easy-rsa repository:  
```
git clone https://github.com/OpenVPN/easy-rsa.git
```
```
cd easy-rsa/easyrsa3
```
b. Initialize the PKI environment:  
```
./easyrsa init-pki
```
c. Create a CA certificate:  
```
./easyrsa build-ca nopass
```
d. Create a server key and certificate:  
```
./easyrsa --san=DNS:server build-server-full server nopass

```
e. Create a client key and certificate:  

```
./easyrsa build-client-full client1.domain.tld nopass
```
f. Copy the certificates and keys to a designated folder:  

```
mkdir ~/custom_folder/
cp pki/ca.crt ~/custom_folder/
cp pki/issued/server.crt ~/custom_folder/
cp pki/private/server.key ~/custom_folder/
cp pki/issued/client1.domain.tld.crt ~/custom_folder
cp pki/private/client1.domain.tld.key ~/custom_folder/
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
**8. Connect Using Client VPN Tool**  
- Open the Client VPN tool and import the configuration file.
- Click on Connect.

**9. Validate Connectivity**  
- Ping the private IP address of the instance.
- SSH into the server  
  ssh -i mykey.pem ec2-user@private-ip

References  
For more detailed information, refer to the AWS Client VPN documentation.  
https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/mutual.html  
https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/cvpn-getting-started.html  
https://docs.aws.amazon.com/vpn/latest/clientvpn-user/client-vpn-connect-windows.html  

That's it!!!  
Now we have successfully setup AWS Client VPN  
