# Linux Assignment 3

# Step 1: Install all necessary software

- a: Update pacman
- b: Install Vim
- c: Install nginx
- d: Start and enable nginx

## a: Update pacman

So Before we begin, we need to make sure we have all the software we need to complete this activity and all that software is up to date

Run the following command to update all currently installed pacman packages before we can install new ones:

```bash
sudo pacman -Syu
```

Press Y then ENTER to confirm installation. 

## b: Install Vim

Vim is the primary editor we will be using to write to files. To install Vim, run the following command: 

```bash
sudo pacman -S vim
```

Press Y then ENTER to confirm the installation

## c: Install nginx

> Nginx is a high-performance web server known for its stability, simplicity, and low resource consumption. It's commonly used to serve web content, handle reverse proxying, load balancing, and more. Nginx is often preferred for its ability to efficiently serve static content and its flexibility in configuring various types of web applications. - Wikipedia
> 

```bash
sudo pacman -S nginx
```

Press Y then ENTER to confirm the installation

d: Start and enable nginx

> systemctl is a command-line utility used for controlling the systemd system and service manager in Linux. It provides a way to manage system services, including starting, stopping, restarting, enabling, disabling, and checking the status of services. Systemd is the default init system for many modern Linux distributions, including Arch Linux, and systemctl is the primary tool for interacting with it. - Wikipedia
> 

Use systemctl to interact with services in Linux. Services are programs that operate in the background to allow you to interact seamlessly with your device.

Command usage is sudo systemctl [option] [target]. In this case, “sudo systemctl start nginx” will start the nginx service. The “enable” causes it to start on boot. Run the following commands:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

To check if this was successful, you can use “sudo systemctl status” to see the status of a service. If it is enabled and active, it will display that in green. Run the following command:

```bash
sudo systemctl status nginx
```

# Step 2: Configuring Nginx to display a Web page

In this step we will be using Nginx to display a web page

## a: Create directories where we will be working.

We will create directories where we will store our files for our project and be the base for our Git repo.

We can use the “mkdir -p” command to create the multiple directories we will need for this project. Run the following command to do:

```bash
# cd into /
cd /
mkdir -p web/html/nginx-2420
```

## b:  **Set Up a Separate Server Block**

> A server block, also known as a virtual host, is a configuration block in Nginx that defines how the server should respond to different requests. Each server block can have its own settings, such as listening port, domain name, root directory, and other configurations. - Wikipedia
> 

We need to create a new server block so we can properly respond to requests for websites. To do this, you must go to the Nginx configuration directory with “cd /etc/nginx“

```bash
# Go to /etc/nginx
cd /etc/nginx
# Create the directories
sudo mkdir sites-available
sudo mkdir sites-enabled
cd sites-available
sudo vim example.conf
```

## example.conf

```bash
server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   /web/html/nginx-2420;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
```

Make sure to wq out of vim.

## c: Link the new configuration file

After creating the 'example.conf' file, we need to create a symbolic link to it in the 'sites-enabled' directory. This will allow Nginx to include this configuration of the server block during startup. Run the following commands:

```bash
# Go back into /etc/nginx
cd ..
# Create the symbolic link
ln -s /etc/nginx/sites-available/example.conf /etc/nginx/sites-enabled/

```

Remember to check your configurations with 'sudo nginx -t'. If the output states that the configuration file test is successful, you can proceed to reload Nginx.

Now you can access the server block from **example.conf** in **nginx.conf**

```bash
#user http;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
....
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    include sites-enabled/*;
}
```

```bash
sudo systemctl restart nginx
```

## d: Serve HTML page using git

cd to web/html/nginx-2420

Then, initialize a Git Repository in this directory with git init. Init is used to create a new empty repository (well, technically, it adds a git file that serves as the foundation for the git repository).

NOTE: We are not adding it to GitHub yet. That’s the next section. This section is creating a local git repo we will put on GitHub later.

```bash
cd /web/html/nginx-2420
sudo git init
```

Use Vim to create a file, index.html to be served:

```bash
sudo vim index.html
```

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
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

Use git add to add the index.html to the staging area Git holds all changes in this staging area until you are ready to commit them later.

```bash
sudo git add index.html
```

Use git commit to finally upload and commit the changes to just this local repo for now:

```bash
sudo git commit -m "Initial commit"
```

If you haven’t set up your git profile yet: use “ **git config --global user.email** "*you@example.com*"” and “**git config --global user.name** "*Your Name*" to set up and link your git profile (It will tell you to do this on your screen if you haven’t already). And then run git commit -m “initial commit”

## e: Connect to GitHub:

Make sure you have an empty repo on GitHub already created and available to use. Then just run the following:

```bash
sudo git branch -M main
```

```bash
sudo git remote add origin https://github.com/yourusername/your-repo.git
```

Replace the url to your new repo’s git file

Finally, push files to the new repo:

```bash
sudo git push -u origin main
```

Once you do this, GitHub will ask you to login. Enter your Username as Normal BUT DO NOT enter your password. GitHub removed support for terminal based password logins like that on August 13, 2021.

Instead, you must go to your GitHub developer settings and generate a new classic access token and use that in place of your password.

Once you have done this, use sudo systemctl reload nginx. Then open your browser and enter in your droplet’s IP address. If you did everything correct, you should see your webpage.
![image](https://github.com/rylanraj/nginx-2420/assets/76143775/6528c75a-40d3-4e7d-8438-5955f7870482)
