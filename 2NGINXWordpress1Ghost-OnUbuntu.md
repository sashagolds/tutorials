# Installing 2 Wordpress (NGINX), plus Ghost on 1 Ubuntu 14.04 VPS (Digital Ocean)
I couldn't find any tutorials ANYWHERE on how to Install 2 Wordpress blogs (not multisite) side by side on NGINX. I also couldn't find anything on Installing Ghost next to an NGINX WordPress — so I had to figure it out on my own. After lots of pain & suffering, I got the process worked out. So for anyone who wants to do the same, Here are the steps I took below. I am by no means knowledgeable in server languages or setup, so anyone please feel free to add their own corrections and clarifications as necessary, or to copy this/add to it on a blog/somewhere online for the benefit of others. No attribution needed... I'd like this to become a shared resource.

0. Set up 3 domains (one for each site) and point them at DO by following this tutorial: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean

1. Follow Tutorial 1 - https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-14-04-lts
1.1 set up 3 root directories in /var/www/ and 3 server blocks with a TEST index.html in /etc/nginx/sites-available/: 1 for each WP install, and 1 for Ghost.
1.2 verify that each is set up correctly and works

2. Follow Tutorial 2 - https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-nginx-on-ubuntu-14-04, doing these steps for each of the 2 WP root directories you set up:
	- step 4
	- step 5 
2.1 You should have already set up 2 server blocks and linked them to your sites-enabled folder in the first tutorial, so you do not have to REDO it for step 5. Instead, you need to use 'sudo nano' to open each of the server blocks you created in tutorial 1 and make them look like the example in step 5, WITH A COUPLE EXCEPTIONS:
	- In your second server block make sure you comment out/delete the line 'listen [::]:80 default_server ipv6only=on;' AND the directive 'default_server' from 'listen 80 default_server;' (as per 'Create the Second Server Block File' from Tutorial 1)
	- you should not need to execute 'sudo ln -s...'' again as you've already done it
	- Make sure you restart nginx and php5 after this is done.

3. Follow Tutorial 3: https://www.digitalocean.com/community/tutorials/how-to-host-ghost-with-nginx-on-digitalocean
3.1. After step 2, use the ls command to verify the ghost files are unzipped into your base root directory for Ghost from tutorial 1 (/var/www/ghost/ or whatever - I got a weird error because I had accidentally moved the files into /var/www/ghost/ghost instead)
3.2. You should already have set up the server block in tutorial 1, so you just need to 'sudo nano' your ghost server block. YOU SHOULD DELETE ALL PRE-EXISTING CONTENT FROM TUTORIAL 1 IN THIS BLOCK, and insert:

```
server {
    listen 0.0.0.0:80;
    server_name *your-domain-name*;
    access_log /var/log/nginx/*your-domain-name*.log;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header HOST $http_host;
        proxy_set_header X-NginX-Proxy true;

        proxy_pass http://127.0.0.1:2368;
        proxy_redirect off;
    }
}
```
as per step 3. Your symlink should already be set up.
3.3. Type nginx -t or sudo nginx -t to make sure your nginx config files have no errors, restart nginx.
	- DO NOT FOLLOW STEP 4
	- Ghost should run if you 'cd' to your Ghost root and type 'npm start'. If it doesn't, there is probably something wrong with your config/server block, or you are not in your Ghost root folder. While Ghost is running navigate to your domain and you should see your ghost website. If this works, use Ctrl+C in the command line to close Ghost and move on.
3.6. Go to: http://support.ghost.org/deploying-ghost/ and scroll down to the heading 'Init Script' and use this instead of step 4. Restart your server with 'sudo reboot'. Ghost should run on its own now.
3.7. Don't forget to CONFIGURE Ghost once it is set up using the config.js in your Ghost root as per http://support.ghost.org/config/

You're done :)
