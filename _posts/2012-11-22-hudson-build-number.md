---
layout: post
title: Attaching hudson build number to filename using maven
---

Hello, I had a task to attach hudson build number to file name (like server-1.3.35.ear), and attach "0" when you build project locally, here is my solution. (I'm using maven for build)
First you need to customize output file name, it's simple (I am using this for building ear so I use ear plugin, It will look similar for jar and war build):
{% highlight xml %}
<artifactid>maven-ear-plugin</artifactid>
 <version>2.3.2</version>
 <configuration>
   <finalname>server-ear-${projectVersion}.${buildNumber}</finalname>
   ...
  </configuration>
</plugin>
{% endhighlight %}