## Index
[Spring MVC](https://github.com/Danny7226/LearningSummary/blob/main/java/README.md#spring-mvc)

[Java memory model](https://github.com/Danny7226/LearningSummary/blob/main/java/README.md#java-memory-model)

[Weaving](https://github.com/Danny7226/LearningSummary/blob/main/java/README.md#weaving)

[Dynamic proxy JDK](https://github.com/Danny7226/LearningSummary/blob/main/java/README.md#dynamic-proxy-jdk)

[Tomcat, java EE, java servlet, and socket](https://github.com/Danny7226/LearningSummary/blob/main/java/README.md#tomcat-java-ee-java-servlet-and-socket)

## Topics
### Spring MVC
* Spring MVC takes in http request in DispatchServlet (single entry point)
* DispatchServlet has WebApplicationContext spawned when server spins up
* DispatchServlet will dispatch requests to individual controller
* DispatchServlet is essentially a java servlet, which opens a java socket bonded to a certain port
* DispatchServlet acts as proxy so that it loads requests to RestController in different url path
* Root context is used by controllers, whereas WebApplicationContext is used by front controller (DispatchServlet)
* RestController returns data in textual format, whereas Controller returns the data to a ViewResolver

### Java memory model
* https://medium.com/platform-engineer/understanding-java-memory-model-1d0863f6d973
* Java source code will be compiled by javac(java compiler) into .class files
* .class files are data in bytecode format in binary. People might think it's hexadecimal, but it's actually not. Hexadecimal representation is just a way of viewing binary data
* Java Runtime Environment (JRE) will take .class files. Classloader loads classes into JVM in runtime
* Classloader doesn't change the format of data, it simply loads files into memory
* Bytecode is platform independent, meaning it can be executed in any compatible JVM platforms. (Windows, Linux, OSX have different implementations of JVM, JVM acts as an abstract layer between bytecode and hardware OS)
* We all know static fields are stored into JVM heap instead of stack (each thread has its own stack) memory
* Even within threads, thread creates local objects, which are only visible to themselves. Note that these local objects also stored in heap instead of stack
* Thread maintains a reference in stack pointing to the actual object in heap
* Java garbage collection scans heap memory and cleans objects that are no longer referenced
* Static fields points to the same reference in heap, and therefore multiple threads accessing same static class/method/fields might cause race condition. However there is no guarantee of the visibility for all threads to see the changes
* java volatile is used to make sure data is written into main memory and visibility to all threads

### Weaving
* Weaving is a technical to manipulate byte-code java classes in JVM
* Compile time weaving happens when javac compiles .java into .class
* Post-compile weaving happens to .class files
* Load-time weaving in AOP happens when java class loader loads classes files into JVM
* Runtime weaving happens in JVM (after class loader has loaded .classes files into JVM). (***Quesiton, relation to java agent?***)
* Spring AOP uses a proxy based runtime weaving, and it uses java reflection fundamentally
* Compile time weaving cannot defer decision in runtime, but is better to debug to know problems fast
* Post-compile weaving works well with 3rd party code, which developers don't have source code of in general
* Load-time weaving introduce extra overhead when JVM/application starts up, or server spins up (as class loader needs to spend extra time to weave when loading classes into JVM)
* However, LTW provides feasibility to decide if/what to weave, and bytecode can be manipulated without change source code
* Load-time weaving keeps source code free of aspect related code (so that LTW are used at certain time not all time, say performance monitoring and debugging runtime) https://github.com/indrabasak/spring-loadtime-weaving-example
* LTW fine-grained control on when and where to apply aspects
* Usually config files are load-time-weaving enabled (***needs to do more research***)

### Dynamic proxy JDK vs CGLib
* Proxy patten use case is for access control, such as auth, throttle, delegation, and not about changing behaviors
* Dynamic proxies differ from static proxies in a way that they do not exist at compile time. Instead, they are generated at runtime by the JDK and then made available to the users at runtime
* Spring AOP weaving use JDK dynamic proxy by default. It creates a proxy implements the interface of the target objects and delegate method calls (this requires target object implements an interface)
* CGlib creates proxy by subclassing (extends). Because of this, class or methods should not be final


### Tomcat, java EE, java servlet, and socket
* Apache tomcat is a web server and java servlet container (servlet engine)
* Apache tomcat is not directly built upon java ee, but java se (standard edition)
* However, most application build web application by deploying WAR (web application archive) onto tomcat servlet engine
* Servlet is simply a java class that deals with http request and gives http response
* Java servlet is not directly communicating with java socket. Instead, tomcat (servlet enginer) directly uses java socket listening to a port to establish network transmission
* Tomcat (servlet contain) manages java servlet lifecycles, dispatch requests into appropriate servlet. Servelet can generate HTML, execute logic, access DB and etc
* Java socket fits into transport layer in OSI model (layer 4), socket is a software object representing an IP + port number
* Port is a logical entity. Port, along with IP, uniquely identifies the destination (endpoint) for a data packets in network