---
title: "Setting up WordPress with Docker"
---

The typical recommendation for playing around with WordPress development locally is
to run a local XAMPP or MAMP stack and manually install WordPress. This process
has been streamlined considerably, but it is still a pain point when you develop
on multiple machines. Using Docker, we can have a portable WordPress installation
that requires no setup other than having Docker installed. This way we can get
up and moving in any new environment quickly and easily.

## Prerequisites

We will need both Docker and docker-compose installed on our system. Please follow
the installation instructions for each below.

* [Docker][docker-install]
* [docker-compose][docker-compose-install]

## Docker

First, create a working directory

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
           -p 8080:80 \
           --link some-mysql:mysql \
           wordpress
{% endhighlight %}

This command will start a container from the `wordpress` image:

* `p`: Forward host port `8080` to container port `80`
* `link`: Link the container named `some-mysql` into the container as `mysql`

If you browse to `localhost:8080`, then you should see the WordPress installation page.

**Note**: Changes to the WordPress installation will persist as long as you do not `docker rm` the
database container.

## Docker Compose

Having to remember and run these commands is a pain. This is where `docker-compose` helps. We can
specify multiple Docker containers to be run as a group. Let's see what this configuration looks
like in a `docker-compose.yml` file.

{% highlight yaml %}
wordpress:
  image: wordpress
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

## Conclusion

With this setup, just pull down the `docker-compose.yml` and run `docker-compose up` on any machine
to get a running WordPress server. Look for a future post about leveraging Docker volumes to
use Docker as a local WordPress server for easy theme development!

[docker-install]: https://docs.docker.com/installation/
[docker-compose-install]: https://docs.docker.com/compose/install/
[data-only-container]: https://docs.docker.com/userguide/dockervolumes/
