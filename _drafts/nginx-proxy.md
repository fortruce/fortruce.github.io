---
title: "Using nginx-proxy to Manage a Docker Architecture"
---

Similarly to the previous post we will start a `nginx-proxy` docker container with
our config folder mapped in as a volume.

{% highlight bash %}
docker run -d -p 80:80 --name proxy -v `pwd`/conf/:/etc/nginx/ -v /var/run/docker.sock:/tmp/docker.sock --restart=always jwilder/nginx-proxy
{% endhighlight %}

* `-p 80:80` - map port 80 b/w host and container
* `--name proxy` - name our container for easier reference
* `-v \`pwd\`/conf/:/etc/nginx/` - use a volume to map our config folder into the container
  * Note: Do not use the `:ro` option as `docker-gen` needs to write to this directory
* `-v /var/run/docker.sock:/tmp/docker.sock` - `docker-gen` needs access to the `docker.sock` to hook
into the events
* `--restart=always` - set the restart option to ensure our container is restarted if it exits or the
host restarts

