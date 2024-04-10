# Linux Assignment 3 part 2

We are going to add a firewall using **ufw**, first install **ufw** with **pacman**

```bash
sudo pacman -S ufw
```

Now enable and start the service with **systemctl**

```bash
sudo systemctl enable --now ufw.service
```

We can check the status of ufw with the command

```bash
sudo ufw status verbose
```

This should be inactive since we just installed ufw for the first time

Make sure to allow ssh so you don’t get blocked out of your droplet

```bash
sudo ufw allow ssh
```

Since we’re working with web servers we also want to allow HTTP, and port 8080 (for our proxy in the later steps)

```bash
sudo ufw allow http
sudo ufw allow 8080
```

Now we need to enable the the firewall

```bash
sudo ufw enable
```

Check the status to see if it worked

```bash
sudo ufw status verbose
```

## Moving the backend bin

We will be using **sftp** to move the provided **hello-server** file into our droplet

Make sure on your **host machine** has the **hello-server** file in your **C:\Users\YOUR-USERNAME**

This is because sftp will look here specifically for the file

Now, start sftp between your host and your droplet

```powershell
sftp  -i .ssh/do-key name@ServerIPAddress #Change this as needed
# Or if you have it configured
sftp archlinux # Replace archlinux with what you put as the host in your SSH config file
```

Once you’ve done this, we need to run the **put**, to put the **hello-server** file in your droplet.

Put it in your **home directory** or you may get a **“Permission Denied”**

```powershell
put hello-server/home/YOUR-USERNAME #Change YOUR-USERNAME
# Exit sftp
exit
```

Now we will move this file to a more logical location, since this file is a backend file for our server, we will move it to **/usr/local/bin/backend**

(Make sure to SSH back into your droplet first

```bash
sudo mv hello-server /usr/local/bin/backend 
sudo chmod u+x /usr/local/bin/backend #It also has to be executable for the next steps
```

## Setting up the service file to start the backend

Now we will write a service file to start the backend

Service files are usually in **/etc/systemd/system**

So this is where we’ll put the file

```bash
sudo vim /etc/systemd/system/backend.service

# Paste the following into vim (Press i to go into insert mode)
[Unit]
Description=Backend Service
After=network.target

[Service]
Type=Simple
#This is where the backend file is located.
ExecStart=/usr/local/bin/backend
Restart=always

[Install]
WantedBy=multi-user.target
# Then, CTRL-C or ESCAPE, followed by :wq to write quit out of the file
```

Using **systemctl** start and enable this new service we made

```bash
sudo systemctl start backend
sudo systemctl enable backend
```

## Setting up the reverse proxy to the backed

Now we will set up a reverse proxy server to the backend binary on our system.

First, we need to vim into the example configuration file we made back in part 1

```bash
sudo vim /etc/nginx/sites-available/nginx-2420/example.conf
```

Then we need to add this code INSIDE the server block

```bash
# add this INSIDE the server block, it defines the proxy settings
location /hey {
		# The 'proxy_set_header' directives are used to pass headers from the original request, to the proxy server.
    proxy_pass http://127.0.0.1:8080;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

}

location /echo {
    proxy_pass http://127.0.0.1:8080;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

}
# Then, CTRL-C or ESCAPE, followed by :wq to write quit out of the file
```

Since we changed an nginx configuration file, we need to test if the changes we made were okay, with the following commands

```bash
sudo nginx -t # This is supposed to say all configuration files OK and test is SUCCESSFUL, if an error occurs redo the steps
sudo systemctl restart nginx # We have to restart since we changed the .conf file
```

Now we will test connecting to the backend, I will be using **Postman** to do so

This will test **YOUR_DROPLET_IP_ADDRESS/hey**, via HTTP GET

The response body should be “Hey there”

![Untitled](https://github.com/rylanraj/nginx-2420/assets/76143775/d417f68a-7a36-4d80-8229-f8b80bf50b34)

This will test **YOUR_DROPLET_IP_ADDRESS/echo**, via HTTP POST

The response body should be 

{
"message": "Hello from your server"
}

![Untitled 1](https://github.com/rylanraj/nginx-2420/assets/76143775/bd2cc52c-840d-4f9d-8a96-877a0f499d9d)

