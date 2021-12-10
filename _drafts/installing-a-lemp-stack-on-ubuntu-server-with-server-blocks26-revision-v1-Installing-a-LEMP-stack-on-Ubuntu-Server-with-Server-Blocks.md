---
id: 53
title: Installing a LEMP stack on Ubuntu Server with Server Blocks
date: 2018-01-03T12:26:14+00:00
author: tom
layout: revision
guid: http://3.8.252.104/2018/01/03/26-revision-v1/
permalink: /?p=53
---
This article will cover installing Nginx (with HTTPS), PHP and Mysql on a Ubuntu 17.10 Server. The first step is to install Ubuntu Server, my installation was done on a virtual machine within Hyper-V however the installation should be the same for pretty much every scenario. SSH was installed when prompted during the installation and LEMP wasn&#8217;t (just so it could be covered in more detail here).

All commands will be performed via SSH however you can just run these directly.

**Install Nginx**

<pre>sudo apt-get install nginx
</pre>

As part of installing nginx you should configure the firewall, see [this link](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04) for help with that. From this point onwards we will assume this has been done correctly.  
**Create new Directory for new site**

My nginx server will host multiple different sites, because of this we will do things slightly different. We will setup virtual hosts for each site so each site needs a directory.

<pre>sudo mkdir -p /var/www/example.com/public_html
</pre>

**Set permissions**

<pre>sudo chown -R tom:www-data /var/www/example.com/public_html</pre>

<pre>sudo chmod 755 /var/www
</pre>

**Create the Page**

<pre>sudo nano /var/www/example.com/public_html/index.html</pre>

<pre>&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;www.example.com&lt;/title&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;h1&gt;Success: You Have Set Up a Virtual Host&lt;/h1&gt;
  &lt;/body&gt;
&lt;/html&gt;
</pre>

**Create virtual host file**

<pre>sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com
</pre>

**Setup virtual host file**

<pre>sudo nano /etc/nginx/sites-available/example.com</pre>

<pre>server {
        listen   80; ## listen for ipv4; this line is default and implied
        #listen   [::]:80 default ipv6only=on; ## listen for ipv6

        root /var/www/example.com/public_html;
        index index.html index.htm;

        # Make site accessible from http://localhost/
        server_name example.com;
}
</pre>

<pre>sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com</pre>

<pre>sudo rm /etc/nginx/sites-enabled/default</pre>

<pre>sudo service nginx restart</pre>

&nbsp;

If all has worked correctly you should be able to navigate to the url specified in your virtual host file and see your page. This is all done over HTTP and ideally we want HTTPS so the next step used LetsEncrypt to setup HTTPS

**Install Certbot**

<pre>sudo add-apt-repository ppa:certbot/certbot</pre>

<pre>sudo apt-get update</pre>

<pre>sudo apt-get install python-certbot-nginx</pre>

**Configure Nginx**

<pre>sudo nano /etc/nginx/sites-available/example.com</pre>

Change to the following (make sure you have an DNS A Record for www. setup)

<pre># Make site accessible from http://localhost/ server_name example.com www.example.com;</pre>

<pre>sudo nginx -t</pre>

<pre>sudo systemctl reload nginx</pre>

**Allow HTTPS in**

<pre>sudo ufw allow 'Nginx Full'</pre>

<pre>sudo ufw delete allow 'Nginx HTTP'</pre>

**Obtain an SSH certificate**

<pre>sudo certbot --nginx -d example.com -d www.example.com</pre>

This should also prompt you for various options, simply select what you want and continue. If everything has gone as planned we should now be using HTTPS (open your site in your browser to confirm)

**Install MySQL**

<pre>sudo apt-get install mysql-server</pre>

You will be prompted for a password so choose something strong. Run the following command to secure the installation

<pre>mysql_secure_installation</pre>

**Install PHP**

<pre>sudo apt-get install php-fpm php-mysql</pre>

Open the following file and modify

<pre>sudo nano /etc/php/7.1/fpm/php.ini</pre>

Uncomment cgi.fix_pathinfo and set to 0, it should look like this

<pre>cgi.fix_pathinfo=0</pre>

Save and exit

<pre>sudo systemctl restart php7.1-fpm</pre>

**Modify Nginx to use PHP**

<pre>sudo nano /etc/nginx/sites-available/example.com</pre>

Add index.php before index.html, add the following location blocks

<pre>location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
</pre>

Save and exit. Verify everything is working with the following command

<pre>sudo nginx -t</pre>

If all is okay then reload nginx

<pre>sudo systemctl reload nginx</pre>

You should now have a functioning LAMP box that supports server blocks.