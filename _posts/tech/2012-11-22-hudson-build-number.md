---
layout: post
category: tech
title: Attaching hudson build number to filename using maven
---

Hello, I had a task to attach hudson build number to file name (like server-1.3.35.ear), and attach "0" when you build project locally, here is my solution. (I'm using maven for build)
First you need to customize output file name, it's simple (I am using this for building ear so I use ear plugin, It will look similar for jar and war build):
{% highlight xml %}
<plugin>
<artifactid>maven-ear-plugin</artifactid>
 <version>2.3.2</version>
 <configuration>
   <finalname>server-ear-${projectVersion}.${buildNumber}</finalname>
   ...
  </configuration>
</plugin>
{% endhighlight %}

Everything that I need is to init __${buildNumber}__ variable with hudson build number when I am using hudson and "0" when I am building project locally. I use profiles for this:
{% highlight xml %}
<profiles>
   <profile>
      <id>default</id>
         <activation>
            <activebydefault>true</activebydefault>
          </activation>
          <properties>
             <buildnumber>0</buildnumber>
           </properties>
   </profile>
    <profile>
      <id>hudson</id>
        <properties>
           <buildnumber>${BUILD_NUMBER}</buildnumber>
        </properties>
    </profile>
</profiles>
{% endhighlight %}
So, when I build locally maven takes default profile because it is "activeByDefault", and I added __-Phudson__ to maven build string in hudson. 

So now it looks like this: __clean install -Phudson__.

__${BUILD_NUMBER}__ - variable that hudson passes to maven, when it starts build.

__-P__ - is maven command line argument for choosing profile.
