
# **How to Deploy NextJS app on VPS Full tutorial**

Hosting our Next JS app on a custom VPS is an option that often gets overlooked. While Vercel is a popular choice, you might not feel comfortable with a managed hosting provider or you may want to test on your own VPS. If you search for tutorials on “deploying Next JS on a VPS,” you’ll find plenty, but none of them seem to be complete and guide you step-by-step from setting up the VPS to making your app public. That’s why I’ve written this post — to provide a comprehensive guide for those who want to host their Next JS app on a custom VPS.

# **Prerequisites**

## 1. VPS with Linux os

## 2. Domain name for our website

# **Getting a VPS**

If you’re looking to host your VPS, there are many options available out there. However, I prefer working with [cherryservers](https://www.cherryservers.com/?affiliate=QHPJ0XXA) due to several reasons. Firstly, you can order a new VPS and pay an hourly rate, which is great for testing purposes. This means that you don’t have to spend a lot of money on a new server just to test something out. You can get a new VPS for almost nothing, at 0.00USD/hr. Additionally, you can use the coupon code “NEWSLETTERGANG10” to get a 10% discount.

# **Getting a domain name**

## There are a lot of domain name providers available, but you can consider using Hostinger.

# **Now let’s start**

## 1. Connect to your VPS through `SSH`

❯ ssh root@your_server_ip

## 2. Install Nginx

❯ sudo apt update && sudo apt upgrade

this may take a moment

❯ sudo apt install nginx

## 3. Now we need to adjust our Firewall

we can see the Firewall app list by using this command

❯ sudo ufw app list

You should get a list of the application like this:

Output  
  
Available applications:  
Nginx Full  
Nginx HTTP  
Nginx HTTPS  
OpenSSH

now we will only need to allow traffic on port 80 and port 443.

You can enable this by typing:

❯ sudo ufw allow 'Nginx HTTP'  
❯ sudo ufw allow 'Nginx HTTPS'

now we need to enable our Firewall by typing:

❯ sudo ufw enable

Let’s verify the change by typing:

❯ sudo ufw status

Output  
  
Status: active  
To Action From  
 - - - - - -   
OpenSSH ALLOW Anywhere  
Nginx HTTP ALLOW Anywhere  
OpenSSH (v6) ALLOW Anywhere (v6)  
Nginx HTTP (v6) ALLOW Anywhere (v6)

Now let’s check with the systemd init system to make sure the service is running by typing:

❯ systemctl status nginx

Our service has started successfully, we can test it by requesting a page from Nginx.

We can visit the Nginx default landing page by entering your VPS IP address into your browser’s address bar:

http://your_vps_ip_address

if you don’t know your IP address you can find it by using [ipinfo.io](https://ipinfo.io/) by typing:

❯ curl ipinfo.io

Output  
{  
  "ip": "your_VPS_ip_addrs",  
  "city": "#########",  
  "region": "#######",  
  "country": "##",  
  "loc": "######,####",  
  "org": "#######",  
  "postal": "####",  
  "timezone": "America/New_York",  
  "readme": "https://ipinfo.io/"  
}

Cool we ended up setting up our Nginx server for now :ppy:

# 4. Amazing now we are ready to config Nginx as a reverse proxy

## - Create a new Nginx configuration file for your Next.js application:

❯ sudo nano /etc/nginx/sites-enabled/nextjs.conf

## - Paste the following configuration, replacing your_domain_name with your domain name.

server {  
    listen 80;  
    server_name your_domain_name www.your_domain_name;  
    location / {  
        proxy_pass http://localhost:3000;  
        proxy_http_version 1.1;  
        proxy_set_header Upgrade $http_upgrade;  
        proxy_set_header Connection 'upgrade';  
        proxy_set_header Host $host;  
        proxy_cache_bypass $http_upgrade;  
    }  
}

Save and close the file. you can save the file by clicking `ctrl+o`

Now let’s remove the default conf file by typing:

❯ sudo rm /etc/nginx/sites-enabled/default

## - Restart Nginx:

❯ sudo service nginx restart

# 5. install NodeJS on our server

we can install NodeJS by typing:

❯ sudo apt-get update && sudo apt-get install -y ca-certificates curl gnupg  
❯ curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg - dearmor -o /etc/apt/keyrings/nodesource.gpg  
❯ NODE_MAJOR=20  
❯ echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list  
❯ sudo apt-get update && sudo apt-get install nodejs -y

We can check if we install Nodejs by typing:

❯ node

Output  
  
Welcome to Node.js v21.1.0.  
Type ".help" for more information.  
>

# 6. Create a new NextJS app

## - Create a file for our app:

❯ mkdir /var/www/nextjs  
❯ cd /var/www/nextjs

## - Create a new nextjs app:

change “myapp” with you app name and you can clone your project but for now, we will only create new app.

❯ npx create-next-app@latest myapp

## - Let’s run the dev environment of the app:

❯ npm run dev

Cool now let’s build our app

## - build next app:

❯ npm build

# 7. install PM2 globally on your VPS:

❯ sudo npm install -g pm2

# 8. Start Next.js application using PM2:

❯ sudo pm2 start npm --name "myapp" -- start

## - start PM2 on boot:

❯ sudo pm2 startup

## - save PM2 processes:

❯ sudo pm2 save

let’s make sure our Nginx server is secure with SSL by setting up free SSL using [LetsEncrypt](https://letsencrypt.org/)

# 9. Instelling Certbot:

## - Making sure the snap core is up to date:

❯ sudo snap install core; sudo snap refresh core

## - Install the certbot package:

❯ sudo snap install --classic certbot

## - Link the certbot command to your path:

❯ sudo ln -s /snap/bin/certbot /usr/bin/certbot

# 10. Obtain SSL certificates through Certbot:

❯ sudo certbot --nginx -d your_domain_name.com -d www.your_domain_name.com

change your_domain_name to your domain.

In conclusion, preparing a server to host a NextJS app can seem like a daunting task, but it can be achieved with the right knowledge and resources. By ensuring that your server is properly set up, connected to your domain, and secured with SSL, you can create a reliable and efficient platform for your app to run on. Don’t forget to also point your domain name to your VPS IP and enjoy the benefits of a well-prepared server. Good luck!

If this guide helped you navigate the world of deploying Next.js apps on a VPS, consider supporting my work by buying me a [coffee](https://www.buymeacoffee.com/zayk)! ☕ Your contribution goes a long way in fueling future tutorials and content. Thank you for being part of the learning journey! 🚀