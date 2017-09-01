---
layout: post
title: How to host multiple websites on a single server using NGINX reverse proxy
---
## Summary - In this tutorial we will see how to host more than one web application on a single server, either on multiple domains or on domain sub folders.

## Objective
How to setup multiple web applications in a single server which are either accessible on separate domains or on separate domain sub folders.
## Description
Sometimes it is required to host more than one web application in a single server. In this tutorial we can achieve that using nginx reverse proxy.

### Example Configuration
We have three web applications hosted on a Linux machine. For this example, NGINX is installed on the same server as these web applications but it could be on a separate server of its own.

Our example web applications are listening on different ports on the same web server. We want to associate different domain names/URLs with these applications. The localhost ports and the external domain names are mentioned in the table below:

| Application | Port | Domain | Remarks |
| -------------- | ----- | ---------- | ---|
| Application1 | 5000 | www.testapp.com | First web application
| Application2 | 5010 | www.testapp.com/reward | Another web application in a sub folder
| Application3 | 5020 | blog.testapp.com | Third webapp on a different domain|

Now we will see how to achieve this kind of setup for our web applications:

### Step One: Install NGINX on your server
NGINX is an open source web server. It is majorly used on UNIX like operating systems, although it could easily be installed on Windows as well.



For this example e are using UBUNTU 16.04 server for NGINX as well as for our web applications. 
NGINX is available in default repositories of most Linux distributions and in Homebrew for Mac OS. 

First we will update the local package and then we will install NGINX using the following commands:

```
$ sudo apt update
$ sudo apt install nginx
```
### Step Two: Configure the firewall
Next we will open the firewall for the required ports so that our web applications are reachable from the outer world.
Since we are using NGINX as the reverse proxy, we will only need to open port 80 (HTTP) or port 443 (HTTPS). NGINX will redirect the traffic to the correct port for the different web applications. 

Keep in mind that in our example NGINX and web applications are installed on the same server. If there is a separate server acting as reverse proxy, we might need to open firewalls for the individual applications as well. 

Steps for configuring firewalls will be different for each operating system and each firewall configuration software. UFW is the default firewall configuration software for Ubuntu and we are using the same in this example. UFW provides a **U**ser **F**riendly **W**ay of configuring the firewall.

1. First execute the following command to turn on the UFW with the default rules:
`sudo ufw enable `

2. Now run the following command to open the required ports:
    ```
    ## To allow ports through the Nginx profile
    # For HTTP
    sudo ufw allow 'Nginx HTTP'
    
    # For HTTPS
    sudo ufw allow 'Nginx HTTPS'
    
    # To allow both HTTP and HTTPS
    sudo ufw allow 'Nginx Full'
    
    # Or to allow particular ports:
    # For HTTP
    sudo ufw allow 80
    
    #For HTTPS
    sudo ufw allow 443
    ```
### Step Three: Check that NGINX web server is working fine
Execute the following command to check that the web server is working fine: `curl localhost`

You should get the default nginx page as the output. 
```
# Command - 
curl localhost

# Output - 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
### Step Four: Setup NGINX reverse proxy to access our different web applications

Now we will configure the NGINX reverse proxy to catch the incoming requests to the web server through different URLs and pass on the request to the correct web application.

It is possible to store the configurations for each virtual host in a separate file, but to keep things simple we are going to edit the default config file of NGINX to setup all the virtual hosts:

```
sudo nano /etc/nginx/sites-available/default
```
Things are quite simple for HTTP connections. For HTTP, this is how config for a single virtual host looks like. Multiple such server blocks could be added in the same config file for each virtual host:
```
server {
    listen 80;

    server_name www.example.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
We will now see what the different blocks in this config file means:
1. Localhost port (NGINX will listen for requests on this port):
    ```
     listen 80;

    ```
1. Domain name :
    ```
     server_name www.example.com;

    ```
    In our example, configuration for Application1 and Application2 will be done in the same server block because both application are to be configured on the same domain. Application2 will be configured to be accessible through a domain sub folder. This is achieved through multiple location blocks as explained in the next point
1. The following location block is instructing NGINX to redirect the incoming requests on the root of the domain specified in the previous step <code>www.example.com</code> to the localhost URL  <code>http://localhost:8080</code>

    <pre>
location / {
        proxy_pass <strong>http://localhost:8080</strong>;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
</pre>
This block could be modified as following to listen on a domain sub path instead for e.g. <code>www.example.com/test</code>
    <pre>
location <strong>/test</strong> {
        proxy_pass <strong>http://localhost:8080</strong>;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
</pre>

As seen above, we can specify multiple location blocks and server blocks to achieve our desired result. Following is a basic configuration file for HTTP connection to allow Application1, Application2 and Application3 to be reached through separate URLs although all the applications are running on the same server. Replace the content of the NGINX sites configuration file `/etc/nginx/sites-available/default` by the following:
<pre>
# For Application1 and Application2
server {
    listen 80;
    server_name  <strong>blog.testapp.com;</strong>
# Application 1 - Port 5000
# URL: www.testapp.com
    location / {
        proxy_pass <strong>http://localhost:5000</strong>;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
# Application 1 - Port 5010
# URL: www.testapp.com/reward
   location <strong>/reward</strong> {
        proxy_pass <strong>http://localhost:5010</strong>;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

# For Application3
server {
    listen 80;
    <strong>server_name blog.testapp.com;</strong>
# Application 3 - Port 5020
# URL: blog.testapp.com
    location / {
        proxy_pass <strong>http://localhost:5020</strong>;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
</pre>

Now restart the NGINX server by executing the following command:
```
sudo systemctl restart nginx
```
To check the status of the nginx server:
```
sudo systemctl status nginx
```
Now all the web applications should be reachable through the different URLs as specified in the following table:
| Application | Port | Domain | Remarks |
| -------------- | ----- | ---------- | ---|
| Application1 | 5000 | www.testapp.com | First web application
| Application2 | 5010 | www.testapp.com/reward | Another web application in a sub folder
| Application3 | 5020 | blog.testapp.com | Third webapp on a different domain|

Configuration of HTTPS will be covered in a separate blog post. URL of the post will be updated here once that post is available.

