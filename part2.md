## Tutorial: Adding Firewall (UFW) and Reverse Proxy with NGINX

We will be setting up a firewall in this tutorial (non-complex) and configure a reverse proxy using nginx service on arch.

### Things-to-do

Before you begin, ensure you have completed the web server deployment setup AND:

- Backend binary (`hello-server`) is placed on your server.
```bash
put "<location of hello-server on host machine>"
```
This will download the hello-server file from the host machine to the VM's user logged in directory.

- Service file (`backend.service`) is created and configured to run the `backend` as a service.
```bash
sudo systemctl enable backend
sudo systemctl start backend
```
These commands will enable backend to run on boot as well as immediately start backend.

- NGINX is installed and configured to serve your domain.
```
Done in previous set up tutorial.
```

- Make relevant directories
```bash
sudo mkdir -p /opt/backend/
```


### Step 1: Set Up Firewall (UFW)

Firewall is a security feature for your computers against "hackers" and malicious attacks using DDos to the system. It can limit who can connect or even communicate to the server.

1. **Install UFW (if not already installed):**

```bash
   sudo pacman -Syu 
   sudo pacman -S ufw
```
- The first command will sync your downloaded services with the database registry and see if anything needs an update. If it does, it will try updating, and will require you to type "yes" as confirmation.
- The second command will just install ufw service after you type "yes" to the prompt.


2. **Allow SSH and enable UFW:** To make things easier for us, we are going to allow SSH from anywhere since this is not high performance firewall, it will still function properly and allow only computers that have the SSH file.

```bash
   sudo ufw allow SSH
   sudo ufw allow 22
```


3. **Allow access to specific ports (80 for HTTP):** Since we are creating a web server we also want to allow http connections

```bash
   sudo ufw allow 80/tcp
   sudo ufw allow http
```

Now we have defined some rules for our firewall, but we haven't turned it on yet. We can turn our firewall on with this command:

```bash
sudo ufw enable
```
- ***NOTE -***  Make sure to allow SSH positively before using this command otherwise you might not be able to log in to your VM ever again.


4. **Verify the firewall settings:** We can check the status of ufw with the command

```bash
   sudo ufw status verbose
```
- Lists all the current allowed types of connection being allowed by the firewall. This makes sure that you can see if they are allowed or not allowed.

### Step 2: Configure NGINX as a Reverse Proxy

1. **Edit NGINX Configuration:**

Modify your NGINX server block configuration (typically found in `/etc/nginx/sites-enabled/2420-nginx`).
For this assignment we are going to use the server  block configured in Assignment 3 part 1.

```bash
sudo vim /etc/nginx/sites-enabled/2420-nginx
```
- ***NOTE -*** This file that will open up should be the symbolic link to a file in /etc/nginx/sites-available directory, hence changes made to this should reflect in the original. If that is not so, you might have configured things incorrectly. Please go over the instructions from the last set-up to confirm.


2. Now edit the file and make changes to the server block.

```nginx
   server {
       listen 80;
       server_name your_server_ip;

       location /api/ {
           proxy_pass http://127.0.0.1:8080/;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
       location / {
		       try_files $uri $uri/ =404; 
       }
   }
```

**Note:** Replace `your_server_ip` with the actual IP address of your Arch server.
- This location /api/  function can be reconfigured to meet any of your needs as demanded, however I will choose to keep it as such in order to deliver a succinct tutorial.

3. **Test NGINX Configuration:**

Ensure the NGINX configuration is valid:

```bash
   sudo nginx -t
```

4. **Reload NGINX:**

```bash
   sudo systemctl reload nginx
```

### Step 3: Test Backend Connection

1. First connect to your server using sftp
```powershell
sftp <hostname>@<ip_Address_of_VM>
```


2. **Transfer the Binary to Your Server:**

   Backend binary (`hello-server`) is placed on your server.
```bash
put "<location of hello-server on host machine>"
```
This will download the hello-server file from the host machine to the VM's user logged in directory.

3. **Transfer files to the folder:**

```bash
sudo mv ~/hello-server /opt/backend/hello-server
```

4. **Edit service files for backend:**
```bash
[Unit]
Description=Backend Service
After=network.target

[Service]
User=your_username
Group=your_group
WorkingDirectory=/opt/backend
ExecStart=/path/to/backend/binary  # In this case, it is /opt/backend
Restart=always

[Install]
WantedBy=multi-user.target
```

5. **Verify Backend Service Status:**

   Check if the `backend` service is running:

```bash
sudo systemctl enable --now backend
sudo systemctl daemon-reload
sudo systemctl restart backend
sudo systemctl restart nginx
sudo systemctl status backend
```

6. **Test Backend Routes:**

   Use `curl` or a similar tool to test the backend routes:

```bash
curl http://127.0.0.1:8080/hey
curl http://127.0.0.1:8080/echo -d "Hello, Backend!"
```

   You should receive responses from the backend service.

### Step 4: Access Backend via NGINX

1. **Access Backend via NGINX:**

   Open a web browser and navigate to `http://your-VM-ip_address'. NGINX should now proxy requests to your backend service.

### Screenshots

Screenshots are as follows:

- Successful backend route tests (`curl` commands)
	- ![[Pasted image 20240410131526.png]]
	- ![[Pasted image 20240410131614.png]]
- Browser access to backend via NGINX (`http://you-vm-ip-address`)
	- ![[Pasted image 20240410132943.png]]
- POSTMAN (Used for different types of get and post and other HTTP requests)
	- ![[Pasted image 20240410140007.png]]
	- ![[Pasted image 20240410140023.png]]
	- ![[Pasted image 20240410140128.png]]
