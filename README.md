# ft_server

Hello everyone.

I apologize for such a long README file, but I wanted to share what I would really have loved to know at the early stage of this project. Unfortunately, at some point, I ended up putting copied / pasted lines of codes to make things work, barely understanding why it worked that way.

# 1. Let's start it with docker / nginx
So first things first.

Today, I think the best way to start this project is to simply run a container with the needed distribution and nginx, just to see how things are set inside.

Let's have a step by step walkthrough:
In your ft_server project directory,

```$> sudo touch Dockerfile```

```$> sudo mkdir srcs```

```$> sudo touch ./srcs/init.sh```

Now that you have these files/dirs, let's edit your Dockerfile :

https://grafikart.fr/tutoriels/dockerfile-636 for FR video tutorial. Our Dockerfile in this step will be almost identical to this !
https://takacsmark.com/dockerfile-tutorial-by-example-dockerfile-best-practices-2018/ for EN tutorial on Alpine distribution. There are many tips in there, and I suggest anyone to watch this if you can understand english.
https://www.youtube.com/watch?v=XiGUu3q2Mwo this video explains various basic dockerfile commands, for those who want further explanations.
If you have watched all three videos, you will probably understand this :

**Dockerfile :**

`FROM debian`

`RUN apt update && apt install -y nginx`

`COPY srcs/init.sh /usr/bin/`

`RUN chmod 755 /usr/bin/init.sh`

`CMD ["init.sh"]`


**init.sh :**

`#!/bin/sh`

`service nginx start`

`bash`

So if we build and run this container like this, ...

`$> (sudo) docker build -t test_img . `

`$> (sudo) docker run --name test_container -it -p 80:80 test_img`

... as you can see, nginx has now started.
If it has not, don't worry, it is probably because nginx was already running on your host machine, using port 80.
To fix this, just stop nginx on your host machine like this

```$> (sudo) service nginx stop```

Then try again, it should work properly, now.

To check if everything is running right, just type localhost or your ip (ifconfig should get your ip for you) in your host's web browser. You should see "Welcome to nginx ... !". 

So now, we are inside the container.
We will inspect how nginx is installed, and which directories are important for us to make our wonderful ft_server project work.

Let's start by moving to the directory in which nginx is installed.

```$> cd /etc/nginx/```

```$> ls```

You will see there are a couple things in there. We will take a look at nginx.conf file.

```$> cat nginx.conf```

almost at the end of the file, you should see 2 lines starting with include ...

`include /etc/nginx/conf.d/*.conf;`

`include /etc/nginx/sites-enabled/*;`

These two directories, are the locations where nginx will look for configuration files.
Apparently, nginx only takes the last line into account. In this case, include /etc/nginx/sites-enabled/*;

So now that we know this, let's move to this location, and find out what's up there.

```$> cd ./sites-enabled/```

```$> ls```

There is a file called default. This file is the file currently being taken by nginx as configuration.
If you want to change the way nginx works, you probably want to change this file.

And here comes the step 2.

--------------------------------------

# 2. Setting up nginx to display an html page.

For this step, we will need 2 more things. An .html file, and a custom nginx configuration file to find it.
I recommend reading this : https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-debian-10

the nginx file will look like this :

**nginx.conf**

`server {`

        listen      80;

        server_name mysite;

        root        /var/www/mysite;

        index       index.html;

        location / {

                try_files $uri $uri/ =404;

        }

`}`

Now we need to copy these files into the container. Let's edit the dockerfile.

**Dockerfile**

`FROM debian`

`RUN apt update && apt install -y nginx`

`COPY srcs/init.sh /usr/bin/`

`COPY srcs/index.html /tmp/`

`COPY srcs/nginx.conf /tmp/`

`RUN chmod 755 /usr/bin/init.sh`

`CMD ["init.sh"]`


Of course, in order for nginx to find the root directory, it has to exist, and nginx must have access rights. We will write that in the script.
Finally, we will ensure that the configuration file is in the right place.

**init.sh**

`mkdir -p /var/www/mysite`

`chown -R www-data /var/www/mysite`

`chmod -R 755 /var/www/mysite`

`mv /tmp/index.html /var/www/mysite/`

`mv /tmp/nginx.conf /etc/nginx/sites-enabled/default`

`service nginx start`

`bash`


With all this set up, you can build and run your container with the same commands as above.
If you see that nginx is up and running in your container, you can go to your web browser, and type your container ip as url.

to get your container ip, type 

`docker inspect test_container`

from a terminal, or type

`ifconfig`

inside your container, and take the eth0 ipv4 adress.

You should see your html page now. **VICTORY :)**

Take note, that although this is working "just fine" , there could be many things to improve.
For example, we just put nginx.conf into /etc/nginx/sites-enabled/ . But we could have put it into /etc/nginx/sites-available/ without replacing the "default" file, and then we could have created a symbolic link in the /etc/nginx/sites-enabled directory, removing the default symbolic link.
Like this :

mv /tmp/nginx.conf /etc/nginx/sites-available/
rm /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/nginx.conf /etc/nginx/sites-enabled/nginx.conf

This way, the default file remains in the sites-available directory and only links are changed, so we can switch between multiple configuration files (we would have to reload/restart nginx to apply changes tho).

-------------------------------------

# 3. Installing phpmyadmin.

Coming soon.
