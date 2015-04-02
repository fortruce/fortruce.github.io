---
title: "Automating Wordpress Theme Development with Docker and Gulp"
---

Wordpress theme development has a lot of moving parts. You have to ensure you have a
proper XAMPP or MAMP stack with a Wordpress installation and proper database setup. You
also have to make sure the theme files you are editing are correctly updated in the
theme directory of your local server. Do you use a CSS preprocessor like SASS or LESS?
Now you have another step to compile those down into CSS and place them in your theme.
After all of this is done, you have to continually refresh your browser so you can see
your changes.
This is ripe for automation!

We can use a combination of Docker, Gulp, and BrowserSync to automate this entire process.
Now, when you save a file it will automatically be moved and/or compiled into the correct
directory, and your browser page will be automatically refreshed so you can see your changes
immediately.

## Prerequisites

We will be leveraging multiple tools throughout this post. We will gloss over some of the
basics of the particular components, so it would help if you were already vaguely familiar
with the individual technologies.
Below is a list of all the prerequisites you will need to have installed on your system and
links to their installation instructions.

* [Docker][docker-install]
* [docker-compose][docker-compose-install]

## Local Wordpress Server with Docker

First, lets automate the setup of our Wordpress Server and Mysql database with Docker.
The usual approach is to setup a local XAMPP or MAMP stack, but then you have to do that
on every machine you develop on. By using Docker you can avoid having to set up a new
stack any time you switch machines - just install docker and you're good to go.

To run WordPress with Docker, we must run two separate containers. The first container will
be our `mysql` database.

{% highlight bash %}
docker run -d --name some-mysql \
           -e MYSQL_ROOT_PASSWORD=password \
           mysql
{% endhighlight %}

This command will start a container from the `mysql` image:

* `-d`: run in daemon mode
* `--name`: name this container `some-mysql`
* `-e`: set the `MYSQL_ROOT_PASSWORD` environment variable to `password`

Next, we will start our WordPress container that runs the actual web server.

{% highlight bash %}
docker run -d --name some-wordpress \
           -v $(pwd)/theme/:/var/www/html/wp-content/themes/theme \
           -p 8080:80 \
           --link some-mysql:mysql \
           wordpress
{% endhighlight %}

This command will start a container from the `wordpress` image:

* `-v`: link the local `theme/` directory into the container's
        `/var/www/html/wp-content/themes/theme` directory.
        The linked directory will mirror all changes from the
        host into the container.
* `p`: Forward host port `8080` to container port `80`
* `link`: Link the container named `some-mysql` into the container as `mysql`

If you browse to `localhost:8080`, then you should see the WordPress installation page.

**Note**: Changes to the WordPress installation will persist as long as you do not `docker rm` the
database container.

Having to remember and run these commands is a pain. This is where `docker-compose` helps. We can
specify multiple Docker containers to be run as a group. Let's see what this configuration looks
like in a `docker-compose.yml` file.

{% highlight yaml %}
wordpress:
  image: wordpress
  volumes:
    - theme/:/var/www/html/wp-content/themes/theme/
  links:
    - db:mysql
  ports:
    - 8080:80

db:
  image: mysql
  volumes_from:
    - dbdata
  environment:
    MYSQL_ROOT_PASSWORD: password

dbdata:
  image: mysql
  entrypoint: /bin/echo
  command: data only container for db
{% endhighlight %}

There is one difference in the `docker-compose.yml` file. Instead of having our database files
reside in the main database container, we create a third [data-only container][data-only-container].
Data only containers are beyond the scope of this post, but I would recommend reading up on them.

**Note**: Now WordPress changes will persist as long as the `dbdata` container is not removed. Removing
and recreating the `db` container will not affect persistence as all the files actually belong to
the `dbdata` container.

With this `docker-compose.yml` file in place, we should be able to run all of our containers
at once with:

{% highlight bash %}
docker-compose up
{% endhighlight %}

If you followed along with the manual containers, then you will have to stop and remove those containers
to free up the ports.

{% highlight bash %}
# THIS WILL REMOVE ALL CONTAINERS BOTH RUNNING AND NOT RUNNING!
docker ps -aq | xargs docker stop | xargs docker rm
{% endhighlight %}


[docker-install]: https://docs.docker.com/installation/
[nodejs-install]: https://nodejs.org/
[docker-compose-install]: https://docs.docker.com/compose/install/
[data-only-container]: https://docs.docker.com/userguide/dockervolumes/
