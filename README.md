## MERN deployment on AWS Cloud

## Project Description
The project is to deploy a Travel Memory application using the MERN stack (MongoDB,  Express.js, React and Node.js)  on AWS EC2 instances using different Application Load Balancers for frontend and backend services, and storing the data in the Mongo Database ensuring scalable architecture.

## Requirements
1.	Mongo DB Connection String
2.	VPC with 2 public and 2 private subnets 
3.	AMI instance with prerequisites applications installed
4.	Two EC2 instances for backend and two EC2 instances for frontend
5.	Target Group and Application Load Balancer for Backend and Frontend
6.	A domain name to host the website on internet

## Steps
1.	Backend configuration <br>
•	Configure backend with two EC2 instances <br>
•	Create a .env file to incorporate database connection details and port information <br>
•	Deploy the backend service and test with individual EC2 instances <br>
•	Create a certificate for backend using AWS Certificate Manager  <br>
•	Create an Application Load balancer with backend EC2 instances <br>
•	Configure the CNAME record to link backend load balancer endpoint [DNS] <br>
2.	Configure frontend with two EC2 instances <br>
•	Update url.js to ensure the front end communicates effectively with the backend <br>
•	Deploy the frontend and test with individual EC2 instances <br>
•	Create a certificate for frontend using AWS Certificate Manager <br> 
•	Create an Application Load Balancer for frontend EC2 instances <br>
•	Configure the CNAME record to link frontend load balancer end point [DNS] <br>
3.	Domain Configuration <br>
4.	Application Testing <br>

## Architecture
<img width="8241" height="2502" alt="image" src="https://github.com/kumar-DevOps/AWS-MERN-App/blob/main/TravelMemory%20Architecture%20Diagram.png" />

**Flow of the application**
 - End user opens the browser and hits https://kumarkanishk.online/
 - The hit goes to DNS (application load balancer end point) managed by web hosting provier (here namecheap.com)
 - The hit is routed to DNS (which is frontend ALB)
 - Frontend ALB listens on https (port 443)
 - ALB does health checks of the frontend EC2 instances
 - ALB distributes the traffic using round-robin / lead outstanding requests
 - Frontend is communicated with backend by backend DNS (in url.js)
 - The request is route to backend ALB
 - Backend ALB listens on HTTPS (port 443)
 - Backend ALB does health check on the backend EC2 instances
 - Backend ALB is communicated to EC2 instances and communicates to database which connect via connection string (in .env)
 - Response flow (reverse path): 
   - MongoDB returns data -> Backend Node.js
   - Backend Node.js returns API response -> Backend ALB
   - Backend ALB -> Frontend browser (via React)
   - React updates UI page is loaded 


## Backend Configuration
### 1.	Create a separate VPC dedicated for this deployment (not mandatory)** <br>
### 2.	Create an EC2 instance**
 - OS: Ubuntu
 - t3.micro
 - network: select created VPC
 - subnet: public subnet
 - auto-assign public IP: Enable
 - Security group: SSH, HTTP, HTTPS, Port 3000 and 3001 enabled<br>

### 3.	Installing applications / packages and setting up reverse proxy using nginx EC2 instance <br>
Run the below unix commands: <br>

Update and upgrade the OS packages
```
> sudo apt update && apt upgrade -y
```
Install and start NGINX
```
> sudo apt install nginx -y
> sudo systemctl start nginx
> sudo systemctl enable nginx
```
Set reverse proxy for individual instance and update the script for filename – default and path - /etc/nginx/sites-available
```
> cd /etc/nginx/sites-available
> sudo nano default
> cat default
server {
    listen 80;
    server_name kumarkanishk.online www.kumarkanishk.online;
    
    location / {
        proxy_pass http://127.0.0.1:3001;
        proxy_http_version 1.1;

        # Preserve client + original host info
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Good defaults
        proxy_connect_timeout 10s;
        proxy_send_timeout    60s;
        proxy_read_timeout    60s;
    }
}
```
Install NodeJS
```
> curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
> sudo apt install -y nodejs
```
**Note:** npm should be installed in the specific path for backend and frontend, hence, we will install for every EC2 instance respectively
Clone the code from GitHub Repository
```
> git clone https://github.com/UnpredictablePrashant/TravelMemory
```
Create a .env file to incorporate database connection details and port information
```
> sudo nano .env
> sudo nginx -t
> sudo systemctl reload nginx
> cat .env
MONGO_URI='mongodb+srv://reshkumara_db_user:xxxxx@cluster0.yddrvfw.mongodb.net/?appName=Cluster0'
PORT=3001
```
### 4. Create an AMI Image
Create an AMI image by navigating on EC2 Instance -> Actions -> Image and templates -> Create image
<img width="468" height="326" alt="image" src="https://github.com/user-attachments/assets/e2004e49-3fdc-46f7-b19a-6e7ef829a050" />

### 5.	Create 3 EC2 instances from the AMI image (in total 4 – 2 for BE and 2 for FE) 
NOTE: For now, create only EC2 instance for another backend server, so, total 2 EC2 instances
<img width="468" height="511" alt="image" src="https://github.com/user-attachments/assets/62cf28d6-c512-4ae2-8378-ae92f02e4445" />

### 6. Install NPM for 2 backend instances and start the backend server
Run the below commands on terminal
```
> cd TravelMemory/backend
> sudo npm install
```
Verify NodeJS and npm versions
```
> node -v
> npm -v
```
Start the backend server
```
> pwd
/home/ubuntu/TravelMemory/backend
> node index.js
http://locahost:3001
```

### 7. Create Target Group (with HTTP) and Application Load Balancer (with HTTPS) protocol and test the Load balancer
<u>Create Target Groups</u>, Navigate to EC2 -> Load Balancing -> Target Groups 
Under Create target group page, select: 
- Target Type: instances
- Target group name: TravelMemory-TG-Backend
- Protocol: HTTP
- Port: 80
- IP Address: IPv4
- VPC: <Select appropriate VPC>
- Protocol version: HTTP1
- Health checks: HTTP
- Click Next
- Under register targets page, select:
- Available instance: <select the two backend instances created>
- Click Include as pending below
- Make sure the ports should be 80
- Click on Next

In Review and create (final) page, click Create target group<br>
<img width="454" height="648" alt="image" src="https://github.com/user-attachments/assets/fe1d162a-edcf-4b33-a98a-8ff37e543f46" /><br>
<img width="451" height="325" alt="image" src="https://github.com/user-attachments/assets/839266aa-bca7-4402-b764-82ee7d66a010" /><br>
<img width="444" height="290" alt="image" src="https://github.com/user-attachments/assets/5efdecdf-dcda-440d-8f0f-e3a5ffbba7e7" /><br>
<img width="462" height="290" alt="image" src="https://github.com/user-attachments/assets/a4ae604c-8c34-4aea-9e22-024ae074c048" /><br>

<u>Create Load Balancer</u>, navigate to EC2 -> Load Balancing -> Load Balancers
Select Application Load Balancer as type and click on create
Under Create Application Load Balancer page, select:
- Load Balancer Name: <Appropriate name>
- Scheme: Internet-facing
- Load Balancer IP address: IPv4
- VPC: <select appropriate VPC>
- Security Groups: <select appropriate security groups>
- Under Listeners and routing:
  - Protocol: HTTPS
  - Port: 443
- Under Default action:
  - Routing action: Forward to target groups
  - Target group: <select recently created target group>
- Under Default SSL/TLS server certificate:
  - Certificate source: From ACM
  - Certificate (from ACM): <select the certificate which is created>
- Click on Create load balancer

**Note:** wait for at least 2-3 minutes to get the status from provisioning to Active

<img width="200" height="648" alt="image" src="https://github.com/user-attachments/assets/346f4696-c68c-4588-8b3a-ff64e4220951" /><br>
<img width="410" height="194" alt="image" src="https://github.com/user-attachments/assets/5703154d-188b-452a-b8de-b1283f513df4" /><br>

Testing Load Balancer:
- Once the ALB is active, copy the dns and paste it any browser. Prefix with https:// and suffix with /hello to the dns url and you should see Hello World

Add CNAME to web hosting provider: Add the dns (backend endpoint) in your name as –
- Type: CNAME
- Host: backend
- Value: dns (without prefix and suffix)<br>
<img width="410" height="289" alt="image" src="https://github.com/user-attachments/assets/625ec287-e567-4165-8d40-e88b108084d6" /><br>
Now, **backend should be accessible at** https://backend.kumarkanishk.online.live

## Frontend Configuration

### 1.	Configure 2 frontend EC2 instances 
 - OS: Ubuntu
 - My AMIs: [select appropriate AMI]
 - Network select created VPC
 - Subnet: public subnet
 - Auto-assign public IP: enable
 - Security group: SSH, HTTP, HTTPS Port 3000, Port 3001

**Note:** Wait for 3-4 minutes for AMI to configuure the instance and install all the applications: NGINX, Reverse Proxy, node, GitHub clone repository <br>
<img width="477" height="466" alt="image" src="https://github.com/user-attachments/assets/dd07348f-6bdf-43b8-bc1b-7b6362a8a3e3" /> <br>

### 2.	Configure URL.js for frontend to connect to the backend service
Copy the backend URL and navigate to /TravelMemory/frontend/src and edit the file url.js and paste the URL
Verify NodeJS and npm versions
```
> nano url.js
> cat url.js
export const baseUrl = process.env.REACT_APP_BACKEND_URL || "https://backend.kumarkanishk.online.live";
```
### 3.	Install npm for 2 frontend instances and start the backend server
Install npm for 2 frontend instances
```
> cd TravelMemory/frontnd
> sudo npm install
```
Verify NodeJS and npm versions
```
> node -v
> npm -v
```
Start the backend server
```
> pwd
/home/ubuntu/TravelMemory/backend
> npm start
```
Test the individual frontend servers:
 - Copy the public IP servers of each and paste it in any browser and test the application
 - Enter a travel memory detail in this page for the all the fields and submit it, it should save in the database

### 4.	Create a certificate on ACM (AWS Certificate Manager) with your domain name
**Note:** There should be a domain registered in any web hosting provider.
For screenshots refer Step 8 in Backend Configuration

On AWS, search ACM -> go to certificate manager -> Request and follow the below steps
<img width="418" height="112" alt="image" src="https://github.com/user-attachments/assets/fb28175d-f49c-4753-80bb-14b41f93b943" /><br>

In Request public certificate page, 
 - Enter the below details:
   - Domain names: full qualified domain name
   - Once filled click on Request
 - It will navigate to a page where it will have CNAME name and CNAME value
 - Add CNAME name and CNAME value in your web hosting provide as CNAME record
 - Wait for some time, ACM should show a message as issued for the certificate

### 5.	Create Target Group (with HTTP) and Application Load Balancer (with HTTPS) protocol and test the Load balancer

**Create Target Groups**, Navigate to EC2 -> Load Balancing -> Target Groups <br>
Under Create target group page, select: Target Type: instances <br>
 - Target group name: TravelMemory-TG-Frontend
 - Protocol: HTTP
 - Port: 80
 - IP Address: IPv4
 - VPC: (Select appropriate VPC)
 - Protocol version: HTTP1
 - Health checks: HTTP
 - Click Next
 - Under register targets page, select:
   - Available instance: (select the two backend instances created)
   - Click Include as pending below
   - Make sure the ports should be 80
•	Click on Next

In Review and create (final) page, click Create target group

**Create Load Balancer**, navigate to EC2 -> Load Balancing -> Load Balancers
Select Application Load Balancer as type and click on create
Under Create Application Load Balancer page, select:
 - Load Balancer Name: Appropriate name
 - Scheme: Internet-facing
 - Load Balancer IP address: IPv4
 - VPC: select appropriate VPC
 - Security Groups: (select appropriate security groups)
 - Under Listeners and routing:
   - Protocol: HTTPS
   - Port: 443
 - Under Default action:
   - Routing action: Forward to target groups
   - Target group: <select recently created target group>
 - Under Default SSL/TLS server certificate:
   - Certificate source: From ACM
   - Certificate (from ACM): <select the certificate which is created>
 - Click on Create load balancer

**Note:** wait for at least 2-3 minutes till it gets provisioning to Active status

**Testing Load Balancer:**
 - Once the ALB is active, copy the dns and paste it any browser. Prefix with https:// to the dns url and you should see Travel Memory application


## Domain Configuration
**Add CNAME records for the domain**
Add the below records:
 - CNAME Record:
   - Type: CNAME
   - Host: www
   - Value: kumarkanishk.online.live<br>
   
In total, there should be 5 CNAME entries for this project:
1.	Domain – host: www and value: kumarkanishk.online.live
2.	Backend certificate: Step 8 in Backend configuration (for backend.kumarkanishk.online.live)
3.	Frontend certificate: Step 4 in Frontend configuration (for www.kumarkanishk.online.live)
4.	Backend DNS endpoint entry
5.	Frontend DNS endpoint entry

**Tip** – if you need to configure IP address for a single instance, create A record and provide the IP Address of EC2 in Value column
<img width="406" height="293" alt="image" src="https://github.com/user-attachments/assets/63cc4def-26be-4c66-8a1e-392e4e6219eb" />
<img width="1536" height="1024" alt="TravelMemory Architecture Diagram" src="https://github.com/user-attachments/assets/daf7199c-079c-417d-b47a-4a0f80c8eb2d" />
<img width="1536" height="1024" alt="TravelMemory Architecture Diagram" src="https://github.com/user-attachments/assets/0a5c9b13-1ba1-4f65-8b1a-0d81416169a3" />
