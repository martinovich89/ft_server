# ft_server

Hello everyone.

I apologize for such a long README file, but I wanted to share what I would really have loved to know at the early stage of this project. Unfortunately, at some point, I ended up putting copied / pasted lines of codes to make things work, barely understanding why it worked that way.

# 1. Let's start it with docker / nginx
So first things first.

Today, I think the best way to start this project is to simply run a container with the needed distribution and nginx, just to see how things are set inside.

Let's have a step by step walkthrough:
In your ft_server project directory,
1. ```$> sudo touch Dockerfile```
2. ```$> sudo mkdir srcs```
3. ```$> sudo touch ./srcs/nginx.conf```
4. Now that you have these files/dirs, let's edit your Dockerfile :
https://grafikart.fr/tutoriels/dockerfile-636 for FR video tutorial. Our Dockerfile in this step will be almost identical to this !
https://takacsmark.com/dockerfile-tutorial-by-example-dockerfile-best-practices-2018/ for EN tutorial on Alpine distribution. There are many tips in there, and I suggest anyone to watch this if you can understand english.
https://www.youtube.com/watch?v=XiGUu3q2Mwo this video explains various basic dockerfile commands, for those who want further explanations.
If you have watched all three videos, you will probably understand this :

Dockerfile :

//++++++++++++++++//

FROM debian

RUN apt update && \
    apt install -y nginx

COPY srcs/init.sh /usr/bin/

RUN chmod 755 /usr/bin/init.sh

CMD ["init.sh"]

//----------------//

init.sh :

//++++++++++++++++//

#!/bin/sh

service nginx start
bash

//----------------//

So if we build and run this container like this, ...

```$> (sudo) docker build -t test_img .```
```$> (sudo) docker run --name test_container -it -p 80:80 test_img```

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
include /etc/nginx/conf.d/\*.conf;
include /etc/nginx/sites-enabled/\*;

These two directories, are the locations where nginx will look for configuration files.
Apparently, nginx only takes the last line into account. In this case, include /etc/nginx/sites-enabled/*;

So now that we know this, let's move to this location, and find out what is there.

```$> cd ./sites-enabled/```
```$> ls```

There is a file called default. This file is the file currently being taken by nginx as configuration.
If you want to change the way nginx works, you probably want to change this file.

And here comes the step 2.

# 2. Setting up nginx to 
