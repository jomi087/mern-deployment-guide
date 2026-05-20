# Deployment Guide — MERN Stack Production Setup

A complete production deployment guide for MERN stack applications covering backend deployment, frontend hosting, MongoDB backup/restore, Nginx reverse proxy configuration, HTTPS setup, PM2 process management, AWS EC2 configuration, security group setup, and production server architecture.

This guide explains how to:

- deploy frontend applications on Vercel
- deploy backend servers on Amazon Web Services EC2
- configure Nginx as a reverse proxy
- secure applications using HTTPS and SSL certificates
- manage Node.js applications using PM2
- restore and backup MongoDB databases
- configure AWS Security Groups and production networking
- understand production-grade backend architecture and deployment workflows

The documentation includes real-world deployment commands, server setup instructions, rate limiting configuration, SSL setup with Certbot, domain configuration, Elastic IP concepts, and security best practices for scalable full-stack application deployment.

---
# MongoDB Backup & Restore Guide

This guide covers the essential commands for creating local backups and restoring them to remote environments like MongoDB Atlas.

---

## 💡 Important Concepts

- **`mongodump`**: High-performance tool for creating a binary export of the contents of a database.
- **`mongorestore`**: Tool for importing content from a BSON HTTP/binary dump created by `mongodump`.
- **Storage**: By default, the `dump/` folder will be created in the directory where you execute the command.

---

## 📂 How to Backup Locally

Use the following syntax to create a backup of a specific local database:

`mongodump --db <DATABASE_NAME>`

Example <br/>
`mongodump --db fixora`<br/>

> This will generate a dump/fixora/ directory containing your backup files.

## How to Restore to MongoDB Atlas (or any remote)

Use this syntax:

`mongorestore --uri="<MONGODB_CONNECTION_STRING>" dump/<DATABASE_NAME>`

Example <br/>
`mongorestore --uri="mongodb+srv://user:password@cluster.mongodb.net" dump/fixora`

> Important: Make sure there's a space between the URI and dump/<DATABASE_NAME>. Also, ensure your MongoDB Atlas allows access from your IP address or is configured to allow access as needed.

## 2. Frontend Deployment (Vercel)

### Steps

1. Connect the GitHub repository to Vercel.
2. Select the frontend project directory (if the repo has multiple apps).
3. Add required environment variables.
4. Click Deploy.

### Notes

- Vercel automatically builds the project.
- After deployment, Vercel provides an HTTPS production URL.
- Every push to the connected branch can trigger a new deployment.

---

## 3. Backend Deployment (AWS EC2)

### Pre-deployment Notes

- Always understand AWS billing before creating resources.
- Set AWS Budget Alerts to avoid unexpected charges.
- Choose the AWS region closest to your users for better latency.

---

### EC2 Setup (Example Configuration)

- Operating System: Ubuntu
- Instance Type: Free-tier eligible (for example, t2.micro)
- Region: ap-south-1 (Mumbai)

### Key Pair Creation

1. Create a new key pair.
2. Enter a name, for example: `my-server-key.pem`
3. Select:
   - Key pair type: RSA
   - Private key format: `.pem`
4. Download and save the `.pem` file securely.

### Important Notes About the `.pem` File

- The `.pem` file gives full access to your server.
- Never share this file with anyone.
- Store it in a safe location on your system.
- Remember the folder where it is saved, because you must run SSH commands from that directory (or reference the full path).

* **Connect to an AWS EC2 Instance via SSH**
  - `ssh -i my-server-key.pem ubuntu@<EC2_PUBLIC_IP>`
  * **Example**
    - `ssh -i fixora-key.pem ubuntu@13.208.108.97`

### Explanation

- `ssh` is the Secure Shell command.
- `-i my-server-key.pem` is the private key file.
- `ubuntu` is the default username for Ubuntu EC2.
- `<EC2_PUBLIC_IP>` is the public IP of the EC2 instance.

### Notes

- Open the terminal from the directory where the `.pem` file is stored, or provide the full path to the key file.
- After running the command successfully, you will be logged into your Ubuntu-based EC2 server.

---

## 4. Server Preparation

- **Update System Packages**
  - `sudo apt update && sudo apt upgrade -y`

- **Add NodeSource Repository**
  - `curl -fsSL https://deb.nodesource.com/setup_<NODE_MAJOR_VERSION>.x | sudo -E bash -`
  * Example
    - `curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -`
    - I have choosed version 20 cz that was the LTS (Long Term Support) version.

- **Install Node.js**
  - sudo apt install -y nodejs

- **Get Backend Code from GitHub**
  - Git is already installed on the EC2 instance, so no need to install it again.
  - `git clone <GIT_REPOSITORY_URL>`
  * **Example**
    - `git clone https://github.com/user/fixora-backend.git`

- **Navigate to Backend Folder (Optional)**
  - my backend folder was inside a parent folder that y
  - `cd <BACKEND_FOLDER>`

- **Install Project Dependencies**
  - `npm install`

- **Create Environment File**
  - `nano .env`
  * **Save and Exit**
    - `CTRL + O` → `ENTER`
    - `CTRL + X`

- **Build Project (TypeScript to JavaScript)**
  - `npm run build`

- **Run the Application**
  - You can run the app directly using:
    - `node your-app.js`

  - However, this approach does not handle:
    - Automatic restarts on crashes
    - SSH disconnections
    - Process monitoring

  - To handle these cases properly, a process manager is required:

### PM2 (Process manager)

- **Run Backend with PM2**
  - **Why PM2?**
    - Keeps app alive after SSH disconnect
    - Auto-restart on crash
    - Zero-downtime restarts

  - **Install PM2**
    - `sudo npm install pm2@latest -g`

  - **Start Application**
    - `pm2 start npm --name "<APP_NAME>" -- run start`
    * This is for one time only next time we jst do line number \*190\*\*

  - **Example**
    - `pm2 start npm --name "fixora-backend" -- run start`
    * The `start` command here calls the `start` script defined in `package.json`

**Useful PM2 Commands** - `pm2 list`

    - **Stop Application**
        - `pm2 stop <APP_NAME | INDEX>`
        - *(You can find the app name or index using `pm2 list`)*

        * **Example**
            - `pm2 stop fixora-backend`

    - **Start Application**
        - `pm2 start <APP_NAME | INDEX>`

    - **Restart Application**
        - `pm2 restart <APP_NAME | INDEX>`

    - **View Logs**
        - `pm2 logs`
        - *(Displays application logs, e.g., `console.log` output)*

- **Backend Deployment Status**
  - The backend project has been deployed successfully.
  - The server is currently running and online.
  - You can verify its status using the process manager (PM2).

- **Connectivity Issue**
  - Currently, the backend is not reachable from outside.
  - This is because the required port is not allowed in the AWS Security Group.

- **Solution**
  - You must update the AWS Security Group to allow inbound traffic on the backend application port.
  - Once the port is allowed, the backend will be accessible and communication will work correctly.

### AWS Security Group (Firewall)

**Inbound Rules**

- **Type:** Custom TCP
- **Port:** `5000` _(backend project port)_
- **Source:** Anywhere (`0.0.0.0/0`)

* **Notes**
  - This acts like a firewall rule that allows traffic **only** on the specified port.
  - Instead of `0.0.0.0/0`, you can restrict access to **your own IP address** so that only that specific IP can access the port.

* **Reminder**
  - This setup is **temporary**.
  - For security reasons, backend internal ports should not be publicly exposed.
  - Later, **Nginx** will be used as a reverse proxy so communication happens through **HTTP (80)** or **HTTPS (443)** instead.

---

**Current Architecture (Not Secure)**<br/>

- Exposing the backend port directly to the internet increases the risk of attacks.

        internet
            ↓
        express App (localhost:<PORT>)

---

### Deployment Status

- The project is now running successfully.
- The frontend runs on the **Vercel** domain.
- The backend runs on the **EC2** server.
- API calls can be made to the EC2 server’s **public IP or domain**

* **Example**
  - `http://<EC2_PUBLIC_IP>:5000/help` (makng an api call via postman)
    ↓
    internet
    ↓
    express App (localhost:5000)

* **Note**
  - This works only if the backend port is publicly exposed via the AWS Security Group.
  - Secure communication between frontend and backend depends on browser rules, CORS, cookies, and overall configuration.

* This setup is **not secure** because it exposes the backend port directly to the internet.

---

**Recommended Architecture (Using Nginx as Middleware)**<br/>

    Internet
    ↓
    Nginx (80 / 443)
    ↓
    Express App (localhost:<PORT>)

- **Nginx** acts as a reverse proxy.
- Only standard web ports (**80 / 443**) are exposed publicly.
- The Express app remains private and accessible only from the server itself.

### Implimenting Nginx

### Why Nginx Is Needed

**Nginx**

- Hides the backend port
- Handles HTTP → HTTPS
- Improves security
- Enables rate limiting
- Prepares the application for scaling
- Supports load balancing

---

### Install Nginx

- `sudo apt install nginx -y`

---

### Nginx Configuration

- **Create Configuration File**
  - `sudo nano /etc/nginx/sites-available/fixora` _(any related name)_

- **Notes**
  - By default, a `default` file already exists inside `sites-available`.
  - We create a **separate configuration file** so that:
    - Each project configuration is clearly identifiable.
    - Adding new projects later is easier.
    - Configuration management remains clean, organized, and understandable.

### Nginx Server Configuration

```nginx
server {
    listen 80;
    server_name <BACKEND_DOMAIN_NAME WWW.DOMAIN NAME>;  //exmpale -> server_name fxora.shop www.fxora.shop;
                                                        # Domain name recommended (IP not recommended)

    location / {
        proxy_pass http://127.0.0.1:<PORT>; // example -> http://localhost:4000;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_cache_bypass $http_upgrade;
    }
}
```

---

### Nginx Rate Limiting Configuration ( if required )

- **Edit Nginx Main Configuration**
  - `sudo nano /etc/nginx/nginx.conf`
  * **Add the following line inside the `http` block**
    - `limit_req_zone $binary_remote_addr zone=global_limit:10m rate=10r/s;`

  * **Example**

  ```nginx
  http {
      ## GLOBAL RATE LIMIT ZONE (per IP)
      limit_req_zone $binary_remote_addr zone=global_limit:10m rate=10r/s;

      # rest of the configuration
      ...
  }
  ```

- **Edit Project Nginx Config**
  - `sudo nano /etc/nginx/sites-available/fixora`
  * **Add Rate Limit Inside `location /` Block** -`limit_req zone=global_limit burst=20 nodelay;`

  * **Example**

  ```nginx
  server {
      listen 80;
      server_name <BACKEND_DOMAIN_NAME>;

      location / {
          limit_req zone=global_limit burst=20 nodelay;

          proxy_pass http://127.0.0.1:<PORT>;
          proxy_http_version 1.1;
          proxy_set_header Host $host;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }
  ```

---

### Enable Nginx Site

- Create a symbolic link to enable the site:
  - `sudo ln -s /etc/nginx/sites-available/fixora /etc/nginx/sites-enabled/`

### Test and Reload Nginx

- Test the configuration:
  - `sudo nginx -t`

- Reload Nginx:
  - `sudo systemctl reload nginx`

### Access Backend via Nginx

- Now that **Nginx is implemented**, the backend is no longer accessed using:
  - `http://<IP_ADDRESS>:5000`

- Instead, you can access the backend using:
  - `http://<IP_ADDRESS>`
  - `http://<DOMAIN_NAME>`

- jst allow the - Port `80` – HTTP in aws security group

* **How This Works**
  - Nginx listens on **port 80**.
  - Incoming requests are forwarded to the backend running on the internal port (e.g., `5000`).
  - The backend port is hidden from the public internet.
  * **Example Flow**

  ```
  http://<EC2_PUBLIC_IP>/help
          ↓
       internet
          ↓
  Nginx (port 80)
          ↓
  Express App (localhost:5000)
  ```

  - API calls (e.g., via Postman) no longer require specifying port `5000`.
  - By default, requests go through **port 80**.
  - note
  * if we try to call `http://<EC2_PUBLIC_IP>:5000/help` it will wrk
    - need to update group security in aws (will do later)

* **Benefits**
  - Improved security
  - Cleaner URLs
  - Easier frontend–backend communicationsss

### How IP Connection Works (Without Direct Nginx Awareness)

- When you access the backend via an **IP address**:
  - The request first reaches the **EC2 instance**.
  - From EC2, it reaches the **Ubuntu server**.
  - The request is then handled by **Nginx**.
  - Nginx decides where to forward the request using:
    - `proxy_pass http://127.0.0.1:<PORT>;`

- This is how Nginx knows where to send the incoming traffic and how it proxies requests to the backend service.

---

### Convert HTTP to HTTPS

#### HTTPS with Certbot

- **Install Certbot and Nginx Plugin**
  - `sudo apt install certbot python3-certbot-nginx -y`

- **Generate and Configure SSL Certificate**
  - `sudo certbot --nginx -d domain_name -d www.domain_name`

---

### Backend API Access (After HTTPS Setup)

- After configuring **HTTPS with Nginx and Certbot**:
  - The API will be accessible over **HTTPS**.

* **Access URLs**
  - `https://<DOMAIN_NAME>`
  - `https://<DOMAIN_NAME>/api-endpoint`

- Accessing via **IP address over HTTPS** is **not recommended**:
  - SSL certificates are issued for **domain names**, not raw IPs.
  - Using an IP may cause browser security warnings.
  - recommending to get domain

* **Result**
  - All API communication is now **encrypted**.
  - Secure frontend–backend communication is enabled.
  - Nginx handles HTTPS and proxies traffic to the backend internally.

* **Update AWS Security Group Rules**
  - **Remove**
    - Port `5000` (backend internal port)

  - **Allow**
    - Port `80` – HTTP
    - Port `443` – HTTPS

---

### Notes

#### UFW (Uncomplicated Firewall) ⚠️

- UFW is an **OS-level firewall** that runs inside Ubuntu.
- It is **optional** when using AWS.
- Commonly used with:
  - Bare-metal servers
  - VPS
  - DigitalOcean droplets
- In AWS, **Security Groups already act as a firewall**, so using UFW is usually **redundant**.

---

### Domain Verification

- You can verify whether a domain is working and correctly mapped to an IP address by using the `ping` command.

* **How to Check**
  - Open your terminal or command prompt.
  - Run:
    - `ping fixora.shop`
    - `ping api.fixora.shop`

- If the **EC2 instance is running** and DNS is configured correctly, you should receive responses.
- In this setup:
  - `fixora.shop` → Frontend
  - `api.fixora.shop` → Backend

---

### Elastic IP vs Public IP

- When an **EC2 instance** is created, AWS automatically assigns a **public IP address**.
- This public IP:
  - Is required to expose Nginx to the internet.
  - **Is not static**.
  - Changes when the instance is **stopped and started**.

- Whenever the public IP changes, it must be updated in:
  - DNS records
  - Client or application configurations

---

### Elastic IP

- AWS provides **Elastic IPs**:
  - Static public IP addresses
  - Intended for **production use**
- ⚠️ Elastic IPs must remain attached to a **running instance**.
  - AWS applies a penalty if they are allocated but not in use.
  - This is because IPv4 addresses are limited resources.

---

### Why Use a Domain Name (Best Practice)

- A domain acts as a **stable abstraction layer** over an IP address.
- Benefits:
  - DNS can be updated if the EC2 public IP changes.
  - Application configuration remains unchanged.
  - Simplifies infrastructure management.
  - Required for **HTTPS (SSL/TLS)**.
  - Clean and professional URLs.

- Although domains have a small cost:
  - They reduce operational overhead.
  - They are considered **best practice** for production deployments using Nginx.

---
