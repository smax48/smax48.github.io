---
layout: post
title: Integrating Java apps with .NET using Azure and Docker
---

The reality today is that a number of great and useful OSS libraries is much more for Java than for .NET.
There is a nice [blog article](http://www.aaronstannard.com/the-profound-weakness-of-the-net-oss-ecosystem/) about .NET OSS community in general.

For example, even a very common task of converting from a docx file to pdf for some reasons has several open source implementations in Java,
in comparison to expensive proprietary solutions in .NET. After some testing, I decided to use the docx4j library. It is quite slow (uses Apache FOP inside),
but results are OK.

But now there is a problem of integration between a web service written in .NET and this Java library.
The service is hosted in MS Azure, so my first idea was to use WebJobs to run Java code, Azure Service bus for communication and Azure storage for data.
I wrote a small and simple Java library that works with Azure queues / blobs and converts incoming docx files to pdf using docx4j.

In general, this works quite good, but it is almost impossible to use free / shared hosting options in Azure for this task - because of memory limits.
I don't know why, but even the "free" tier has more memory available than the "shared" one (1GB vs 512MB per hour).
Because the expected load on this conversion process is not very high, the next available option (dedicated virtual server) is a bit too pricey.

And again, the existing documentation regarding the usage of WebJobs with Java (including possible deployment options etc) is very scarce.

So my next idea was to play with the Docker technology. The one benefit of this approach is that you are not limited to the expensive Azure hosting,
but instead can use a number of good (and cheap) options like Digital Ocean etc.
It turned out to be extremely easy to set up a Docker image for my task.

The resulting Dockerfile is very simple and uses an official java:8 image.

```
FROM java:8

RUN \
	mkdir -p /opt/lib && \
	cd /opt/lib && \
	wget <url to Docx2Pdf_jar.zip file with jars> && \
	unzip Docx2Pdf_jar.zip

WORKDIR /opt/lib/Docx2Pdf_jar

ENTRYPOINT ["java", "-Xmx512M", "-Djava.net.preferIPv4Stack=true"]

CMD ["-jar", "Docx2Pdf.jar"]
```

After publishing this file to the Docker Hub, it is now possible to pull and use this image in any location.
It can be run as a background process and configured to start automatically with the host OS (so reboots are not a problem).

Now I am waiting for the ASP.NET 5 release (RC is expected in November) -
with .NET Core running in a Docker container on Linux we can finally have a lot more choices for cheap hosting options for .NET.
(Of course I know about Mono, but without official support from Microsoft it was always lagging behind).
