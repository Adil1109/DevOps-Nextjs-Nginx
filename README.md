### How to Point Domain and Deploy NextJS Project using Github on Nginx Remote Server or VPS

- Get Access to Remote Server via SSH

```sh
Syntax:- ssh -p PORT USERNAME@HOSTIP
Example:- ssh -p 1034 root@112.42.32.22
```

- Verify that all required softwares are installed

```sh
nginx -v
node -v
npm -v
git --version
pm2 --version
```

- Install Software (If required)

```sh
sudo apt install nginx
sudo apt install git
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```

- Install PM2 (If required)

```sh
sudo npm install -g pm2@latest
```

- Add PM2 Process on Startup

```sh
sudo pm2 startup
```

- Save pm2 config

```sh
pm2 save
```

- Verify Nginx is Active and Running

```sh
sudo service nginx status
```

- Verify Web Server Ports are Open and Allowed through Firewall

```sh
sudo ufw status verbose
```

If you get Inactive run these commands:
```sh
sudo ufw enable
sudo ufw allow OpenSSH
sudo ufw allow 2222/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 3306/tcp
sudo ufw allow 21/tcp
sudo ufw reload
sudo systemctl enable ufw
```

- Exit from Remote Server

```sh
exit
```

- Login to Your Domain Provider Website
- Navigate to Manage DNS
- Add Following Records:

| Type | Host/Name | Value                   |
| :--: | :-------: | :---------------------- |
|  A   |     @     | Your Remote Server IP   |
|  A   |    www    | Your Remote Server IP   |
| AAAA |     @     | Your Remote Server IPv6 |
| AAAA |    www    | Your Remote Server IPv6 |

- Copy Project from Local Machine to Remote Server or VPS. There are two ways to do it:-
  1. Using Command Prompt
     - On Local Windows Machine Make Your Project Folder a Zip File using any of the software e.g. winzip
     - Open Command Prompt
     - Copy Zip File from Local Windows Machine to Linux Remote Server
     ```sh
     Syntax:- scp -P Remote_Server_Port Source_File_Path Destination_Path
     Example:- scp -P 1034 purpledrafts.zip root@112.42.32.22:
     ```
     - Copied Successfully
     - Get Access to Remote Server via SSH
     ```sh
     Syntax:- ssh -p PORT USERNAME@HOSTIP
     Example:- ssh -p 1034 root@112.42.32.22
     ```
     - Unzip the Copied Project Zip File
     ```sh
     Syntax:- unzip zip_file_name
     Example:- unzip purpledrafts.zip
     ```
  2. Using Github
     - Open Project on VS Code then add .gitignore file (If needed)
     - Push your Poject to Your Github Account as Private Repo
     - Make Connection between Remote Server and Github Repo via SSH Key
     - Generate SSH Keys
     ```sh
     Syntax:- ssh-keygen -t ed25519 -C "your_email@example.com"
     ```
     - If Permission Denied then Own .ssh then try again to Generate SSH Keys
     ```sh
     Syntax:- sudo chown -R user_name .ssh
     Example:- sudo chown -R root .ssh
     ```
     - Open Public SSH Keys then copy the key
     ```sh
     cat ~/.ssh/id_ed25519.pub
     ```
     - Go to Your Github Repo
     - Click on Settings Tab
     - Click on Deploy Keys option from sidebar
     - Click on Add Deploy Key Button and Paste Remote Server's Copied SSH Public Key then Click on Add Key
     - Clone Project from your github Repo using SSH Path It requires to setup SSH Key on Github
     ```sh
     Syntax:- git clone ssh_repo_path
     Example:- git clone git@github.com:AdilTestDev/purpledrafts.git
     ```
- Create Virtual Host File

```sh
Syntax:- sudo nano /etc/nginx/sites-available/your_domain
Example:- sudo nano /etc/nginx/sites-available/purpledrafts.com
```

- Write following Code in Virtual Host File

```sh
server{
    listen 80;
    listen [::]:80;
    server_name your_domain www.your_domain;
    location / {
       proxy_pass http://localhost:3000;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection 'upgrade';
       proxy_set_header Host $host;
       proxy_cache_bypass $http_upgrade;
    }
}
```

- Enable Virtual Host or Create Symbolic Link of Virtual Host File

```sh
Syntax:- sudo ln -s /etc/nginx/sites-available/virtual_host_file /etc/nginx/sites-enabled/virtual_host_file
Example:- sudo ln -s /etc/nginx/sites-available/purpledrafts.com /etc/nginx/sites-enabled/purpledrafts.com
```

- Check Configuration is Correct or Not

```sh
sudo nginx -t
```

- Install Dependencies

```sh
cd ~/project_folder_name
npm install
```

- Create Production Build

```sh
npm run build
```

- Create pm2 config File inside project folder

```sh
nano ecosystem.config.js
```

- Write below code in ecosystem.config.js file

```sh
module.exports = {
  apps : [
      {
        name: "myapp",
        script: "npm start",
        port: 3000
      }
  ]
}
```

- Restart Nginx

```sh
sudo service nginx restart
```

- Start NextJS Application using pm2

```sh
pm2 start ecosystem.config.js
```

- Save PM2 Process

```sh
pm2 save
```

- Check PM2 Status

```sh
pm2 status
```

- Now you can make some changes in your project local development VS Code and Pull it on Remote Server (Only if you have used Github)
- Pull the changes from github repo

```sh
git pull
```

- Create Production Build

```sh
npm run build
```

- Reload using PM2

```sh
pm2 reload app_name/id
```

##

### How to Enable HTTPS in Your Domain Hosted on Linux Remote Server or VPS

#### Let's Encrypt is a non-profit certificate authority run by Internet Security Research Group that provides X.509 certificates for Transport Layer Security encryption at no charge.

#### Certbot is a free, open source software tool for automatically using Let’s Encrypt certificates on manually-administrated websites to enable HTTPS.

- To Access Remote Server via SSH

```sh
Syntax:- ssh -p PORT USERNAME@HOSTIP
Example:- ssh -p 1034 root@112.42.32.22
```

- Install Certbot and it’s Nginx plugin

```sh
sudo apt install certbot python3-certbot-nginx
```

- Verify Web Server Ports are Open and Allowed through Firewall

```sh
sudo ufw status verbose
```

- Obtain an SSL certificate

```sh
sudo certbot --nginx -d your_domain.com -d www.your_domain.com
```

- Check Status of Certbot

```sh
sudo systemctl status certbot.timer
```

- Dry Run SSL Renewal

```sh
sudo certbot renew --dry-run
```

Redirect user to www domain to non-www domain

return 301 https://purpledrafts.com$request_uri;

server {
...
client_max_body_size 100M;
...
}

### How to Automate NextJS Project Deployment using Github Action

- On Your Local Machine, Open Your Project using VS Code or any Editor
- Create A Folder named .scripts inside your root project folder e.g. purpledrafts/.scripts
- Inside .scripts folder Create A file with .sh extension e.g. purpledrafts/.scripts/deploy.sh
- Write below script inside the created .sh file

```sh
#!/bin/bash
set -e

echo "Deployment started..."

# Pull the latest version of the app
git pull origin main
echo "New changes copied to server !"

echo "Installing Dependencies..."
npm install --yes

echo "Creating Production Build..."
npm run build

echo "PM2 Reload"
pm2 reload app_name/id

echo "Deployment Finished!"
```

- Go inside .scripts Folder then Set File Permission for .sh File

```sh
git update-index --add --chmod=+x deploy.sh
```

- Create Directory Path named .github/workflows inside your root project folder e.g. purpledrafts/.github/workflows
- Inside workflows folder Create A file with .yml extension e.g. purpledrafts/.github/workflows/deploy.yml
- Write below script inside the created .yml file

```sh
name: Deploy

# Trigger the workflow on push and
# pull request events on the main branch
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

# Authenticate to the the server via ssh
# and run our deployment script
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          port: ${{ secrets.PORT }}
          key: ${{ secrets.SSHKEY }}
          script: "cd ~/project_folder_name && ./.scripts/deploy.sh"
```

- Go to Your Github Repo Click on Settings
- Click on Secrets and Variables from the Sidebar then choose Actions
- On Secret Tab, Click on New Repository Secret
- Add Four Secrets HOST, PORT, USERNAME and SSHKEY as below

```sh
Name: HOST
Secret: Your_Server_IP
```

```sh
Name: PORT
Secret: Your_Server_PORT
```

```sh
Name: USERNAME
Secret: Your_Server_User_Name
```

- You can get Server User Name by loging into your server via ssh then run below command

```sh
whoami
```

- You can get Server PORT by loging into your server via ssh then run below command

```sh
cat /etc/ssh/sshd_config | grep Port
```

- Generate SSH Key for Github Action by Login into Remote Server then run below Command

```sh
Syntax:- ssh-keygen -f key_path -t ed25519 -C "your_email@example.com"
Example:- ssh-keygen -f ~/.ssh/githubaction_ed25519 -t ed25519 -C "githubactionautodep"
```

- Open Newly Created Public SSH Keys then copy the key

```sh
cat ~/.ssh/githubaction_ed25519.pub
```

- Open authorized_keys File which is inside .ssh/authroized_keys then paste the copied key in a new line

```sh
cd .ssh
nano authorized_keys
```

- Open Newly Created Private SSH Keys then copy the key, we will use this key to add New Repository Secret On Github Repo

```sh
cat ~/.ssh/githubaction_ed25519
```

```sh
Name: SSHKEY
Secret: Private_SSH_KEY_Generated_On_Server
```

- Commit and Push the change to Your Github Repo
- Get Access to Remote Server via SSH

```sh
Syntax:- ssh -p PORT USERNAME@HOSTIP
Example:- ssh -p 22 root@112.42.32.22
```

- Pull the changes from github just once this time.

```sh
cd ~/project_folder_name
git pull
```

- Your Deployment should become automate.
- On Local Machine make some changes in Your Project then Commit and Push to Github Repo It will automatically deployed on Live Server
- You can track your action from Github Actions Tab
- If you get any File Permission error in the action then you have to change file permission accordingly.
- All Done
