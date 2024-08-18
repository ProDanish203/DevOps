# Deployment

## Getting started

### Creating and using SSH key
```
# Create an ssh key
ssh-keygen -t rsa -b 4096 -C "github-actions-deploy" -f ~/.ssh/github_actions_deploy -N ""

# Login into your VPS
ssh -i ~/.ssh/PRIVATE_KEY_FILE_NAME USERNAME@SERVER_IP_ADDRESS
```

### Update your system
```
sudo apt update
sudo apt upgrade
```

### Create a non-root User
```
useradd -m -s /bin/bash username
groups username

# This will add the user to the sudoers group, giving them the ability to run commands with sudo privileges.
usermod -aG sudo username 

# Create password for the created user
sudo passwd username
```

### Install Node.Js using NVM

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# Install Nodejs using NVM
nvm install --lts
```

### Install git and Github CLI (Optional)
```
apt install git 
apt install gh

# Login with github
gh auth login
```

### Install pm2 to run the application in backgroud
```
npm i -g pm2
pm2 start npm --name "name" -- start
pm2 list
pm2 logs name
pm2 delete all

// To test our backend
curl http://localhost:8000
```

### Getting started with Nginx
```
apt install nginx

# Start and Enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# some common directives
/etc/nginx/sites-available
/etc/nginx/sites-enabled
/var/www/html

# Some other commands
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl reload nginx
```
- Navigate to `/etc/nginx/sites-available`
- Create a file named after your IP-Address with an extension of .conf Like this: `192.0.0.0.conf`
```
# remove default file
rm default
# create a new file api.conf
Use nano as text editor by:
sudo nano 192.0.0.0.conf

server {
 listen 80;
 server_name _;

 location / {
    proxy_pass http://localhost:3000;
    limit_req zone=mylimit burst=20 nodelay;
    try_files $uri $uri/ /index.html =404;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    #Important For Cookies
    proxy_set_header X-Forwarded-Host $host;
    proxy_cookie_path / "/; Secure; HttpOnly; SameSite=None";
 }
 # If you dont have a domain name, just add like this, otherwise create separate files on the name of their domains
 location /api {
    proxy_pass http://localhost:8000;
    limit_req zone=mylimit burst=20 nodelay;
    try_files $uri $uri/ /index.html =404;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    #Important For Cookies
    proxy_set_header X-Forwarded-Host $host;
    proxy_cookie_path / "/; Secure; HttpOnly; SameSite=None";
 }
}
```
### Add Rate Limitting in Nginx
- Edit the file `sudo nano /etc/nginx/nginx.conf`
```
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;

    ...
}
```

### Create a symbolic link
- Navigate to `/etc/nginx/sites-enabled`

```
# Remove default file
rm default
# Create a symbolic link of the file we created.

ln -s ../sites-available/api.conf
```
- Test and Reload Nginx
```
sudo nginx -t
sudo systemctl reload nginx
```

## Ci/CD Pipeline
- Follow the steps that github action running provides with any user other than root
### Give access to user in that folder
```
sudo chmod -R 777 . 
```
- Follow the instructions and a runner will be added in your git repository with an offline status
### Activate the runner
```
sudo ./svc.sh install
sudo ./svc.sh start
```
- It will now show status of `Idle`
- Start the services with pm2 in the _work folder
- Remove password required for some scripts of the runner user

```
sudo visudo -f /etc/sudoers.d/username

your_runner_username ALL=(ALL) NOPASSWD: /usr/sbin/service nignx restart
```
- Create secret variables in your github repository and create a `deploy.yaml` yaml file in the root directory of your application in the folder named as `.github/workflows`
```
name: Continous Deployment

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build-and-deploy:
    runs-on: self-hosted
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Setup Node.js 22.2.0
        uses: actions/setup-node@v2
        with:
          node-version: "22.2.0"

      - name: Install PM2 globally
        run: npm install -g pm2

      - name: Create server .env file
        run: |
          cd server
          echo "${{ secrets.SERVER_ENV }}" > .env

      - name: Create client .env file
        run: |
          cd client
          echo "${{ secrets.CLIENT_ENV }}" > .env

      - name: Install dependencies and build
        run: |
          cd server
          npm ci
          cd ../client
          npm ci
          npm run build
      - name: Start server and client
        run: |
          pm2 restart server
          pm2 restart client
      - name: Reload Nginx
        run: sudo service nginx restart
```


## Configuring the domain
- Add the two A records
- 1 with @
- 1 with www.
- Create a file at `/etc/nginx/sites-available` by the name of your domain i.e `abc.com.conf`

```
cp api.conf abc.com.conf

nano abc.com.conf

server {
        listen 80;
        server_name <domain> www.<domain>;
        
        location / {
                try_files $uri $uri/ /index.html;
        }
}
```
- Navigate to `/etc/nginx/sites-enabled`
```
sudo ln -s ../sites-available/abc.com.conf

sudo systemctl restart nginx
```
- Domain will be live after all these steps


### Add SSL Ceritificate
```
sudo apt install certbot python3-certbot-nginx

sudo certbot --nginx -d abc.com -d www.abc.com
```

### Usefull links
- https://thapatechnical.shop/blogs/host-a-mern-stack-app-on-a-vps
- https://docs.chaicode.com/server-setup/