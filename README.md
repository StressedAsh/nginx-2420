# Step 1 - Setting up necessary file/folders:

### 1. Log into your droplet -

```
ssh -i .ssh/your_ssh_name user_name@droplet_ip_address
```

### 2. Install nginx by -

nginx is the application that will deploy the server for your need.

```bash
sudo pacman -S nginx
```

### 3. Install Vim by -

Vim is your code editor for this tutorial.

```bash
sudo pacman -S vim
```

### 4. Create required directory in the home folder -

Execute these one by one.

```bash
cd ~
sudo mkdir -p /web/html/nginx-2420
```

cd ~ will navigate your current working directory to the user specific home directory.
sudo mkdir -p will help you create the directory that you need and all the subfolders in one go in the root folder, which is also why you need sudo since normal users don't have permissions to make changes to the root folder.

# Step 2 - Configuring the nginx files

### 1. Creating a configuration file in the required folder -

This will get you into the folder you need to be in to create the valid configuration file for the web server.

```bash
cd /etc/nginx/
```

Create folder called sites-available here by

```bash
sudo mkdir -p sites-available
```

Create folder called sites-available here by

```bash
sudo mkdir -p sites-enabled
```

This will create AND open the new configuration file in that directory that will be required for the server to run. This will also open the file with root permission since normally a user won't be allowed to make changes to this file.

```bash
sudo vim sites-available/nginx-2420.conf
```

### 2. Paste the config code into the opened file -

This is the new configuration details that your server will be using, or more specifically, nginx will be using going forward.

```bash
server {
    listen 80;
    server_name your_droplet_ip;

    root /web/html/nginx-2420;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Be sure to change the "server_name" and "your_droplet_ip" to the name and ip address of your digital ocean droplet in order for this to work. Put them in as is, no need for quotation or other symbols.

As soon as you are done doing that, save and close the file by pressing "Esc" and typing in ":wq"
without the quotations and pressing enter. Now your file is saved!

### 3. Use the following command -

This will restart nginx server

```bash
sudo systemctl restart nginx
```

You can also check if it restarted properly by executing this command

```bash
sudo systemctl status nginx
```

And a more verbose log by using

```bash
journalctl -u nginx
```

Now that the service has restarted properly, lets go back to our home directory by using

```bash
cd ~
```

### 4. Create and open the new HTML file -

Use the following command to create and open the file

```bash
sudo vim /web/html/nginx-2420/index.html
```

Copy and paste the following HTML for the webserver display -

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>2420</title>
    <style>
      * {
        color: #db4b4b;
        background: #16161e;
      }
      body {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100vh;
        margin: 0;
      }
      h1 {
        text-align: center;
        font-family: sans-serif;
      }
    </style>
  </head>
  <body>
    <h1>All your base are belong to us</h1>
  </body>
</html>
```

Make any necessary changes you feel to this file in order to change the display result for the server.

As soon as you are done, save and close the file by pressing "Esc" and typing in ":wq"
without the quotations and pressing enter. Now your file is saved!

### 5. Create a symbolic link to enable the server block.

```bash
sudo ln -s /etc/nginx/sites-available/nginx-2420.conf /etc/nginx/sites-enabled/nginx-2420
```

The symbolic link to enable the server block must be located in the sites-enabled directory within the Nginx configuration directory (/etc/nginx/sites-enabled).

### 6. Check for any syntax errors -

This will provide you a meaningful error message as to what might be wrong with your code.

```bash
sudo nginx -t
```

If it says success for the configuration file and syntax, then you are all good to go! Now reload nginx once again.

```bash
sudo systemctl reload nginx
```

### 6. Open the original config file -

Execute the following command to open the original config file

```bash
sudo vim /etc/nginx/nginx.conf
```

Now copy and paste this code snippet in the conf file, in the https function block

```bash
include /etc/nginx/sites-enabled/*;
```

### 7. Put the ip-address of your droplet in your browser and go to it

![[Pasted image 20240403131554.png]]
This is what it should look like.
