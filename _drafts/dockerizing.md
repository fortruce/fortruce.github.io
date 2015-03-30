---
title: "Docker and Nginx: A Hello World"
---

Lets pull the base nginx docker image.

{% highlight bash %}
docker pull nginx
{% endhighlight %}

We are going to maintain our own git repository of our html (typically `/var/www`) directory and our nginx config files. We will pull the default nginx config as a base. We can utilize the `docker cp` command to copy those out of the base image.

{% highlight bash %}
docker run --name basenginx
docker cp basenginx:/etc/nginx/ .
{% endhighlight %}

Trim this down and just keep the following files:

{% highlight bash %}
nginx.conf
mime.types
conf.d/default.conf
{% endhighlight %}

From now on we will assume the above files reside in the directory `conf`.

Lets also make a simple `index.html` page to serve up as our dummy static content.
Alongside the `conf` directory make a `www` directory with the file `index.html`:

{% highlight html %}
<!DOCTYPE html>
<html>
  <body>
  Hello World!
  </body>
</html>
{% endhighlight %}

Now use the nginx docker image to make a running container. We will map
volumes for both the `www` and `conf` directories so that the container will
use the files from our local filesystem. This way we can update the files locally,
and the container will see the new files without any restarts.

**NOTE**: There appears to be a bug with volumes, nginx, and boot2docker (the OS X docker environment) that make this step not work. [Here is an open ticket.][boot2docker-bug] I sidestepped this issue by ssh-ing into a DigitalOcean droplet running Ubuntu.

{% highlight bash %}
docker run --name mynginx -P -d -v `pwd`/www:/usr/share/nginx/html:ro -v `pwd`/conf:/etc/nginx:ro nginx
{% endhighlight %}

This starts our container:
* `--name mynginx` - name the container `mynginx`
* `-P` - map ports b/w the container and our local system
* `-d` - run in daemon mode
* `-v` - map the two volumes `:ro` = read only inside the container
  * I use `\`pwd\`` here to inject our current working directory since volume
    mapping requires absolute paths.

Now, lets ensure our container is actually running by listing the running containers.

{% highlight bash %}
docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                                           NAMES
5e7a307cbca6        nginx:latest        "nginx -g 'daemon of   19 minutes ago      Up 19 minutes       0.0.0.0:49155->443/tcp, 0.0.0.0:49156->80/tcp   mynginx
{% endhighlight %}

If you don't see the new `mynginx` container running, then you can attempt to find
the issue by checking the logs of the container. Below I accidentally messed up the
volume mapping so it wasn't finding my config files.

{% highlight bash %}
docker logs mynginx
2015/03/30 00:47:23 [emerg] 1#0: open() "/etc/nginx/nginx.conf" failed (2: No such file or directory)
nginx: [emerg] open() "/etc/nginx/nginx.conf" failed (2: No such file or directory)
{% endhighlight %}

Otherwise, you should see the `mynginx` container running! Lets list the ports that
it mapped so we can visit it in the web browser.

{% highlight bash %}
docker port myngix
443/tcp -> 0.0.0.0:49155
80/tcp -> 0.0.0.0:49156
{% endhighlight %}

It will have mapped both port 80 and 443, but only port 80 will actually be serving
connections with our base config. Now, point your browser at `localhost:49156` (**replace with your port**).

{% highlight bash %}
root@Apps:~# curl localhost:49156
<!DOCTYPE html>
<html>
  <body>
  Hello World!
  </body>
</html>
{% endhighlight %}

Now, since we used volumes to control our `www`, we can make changes to
the `index.html` file and see them live by refreshing the page.

To do live updates of the `config` files, you have to modify the config files linked
in the volume and then send a `HUP` signal to the nginx container.

{% highlight bash %}
docker kill -s HUP mynginx
{% endhighlight %}

This will cause nginx to do a hot reload of the `config` files.

## Up Next

This is the first step in getting my VPS set up the way I want. Ultimately, I want
the `nginx` container to be a reverse proxy. It will proxy requests by subdomain to
different web applications running in their own isolated containers. Also, I want to
serve two different domains with this one VPS so it will have to proxy incoming
requests to `example.com` to a separate Wordpress container.

[boot2docker-bug]: https://github.com/boot2docker/boot2docker/issues/652
