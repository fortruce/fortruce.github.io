---
title: "Automating Deployment: The Beginning"
---

I have been working on revamping my blog and portfolio as I get into doing more web
development in my free time. I just finished migrating my blog from Ghost to Jekyll, but
my portfolio is currently empty. I decided to transform my old Ghost VPS to host many
mini portfolio applications. I chose this as a good opportunity to get some experience with
Docker for isolation and automation of deployments.

## Deploying Hello World

{% highlight bash %}
#!/bin/sh

rm -rf hello-world
git clone https://github.com/fortruce/hello-world.git

cd hello-world

docker ps -f image=hello-world -q | xargs docker rm -f

docker build -t hello-world .

docker run -d -p 4321:4321 hello-world
{% endhighlight %}
