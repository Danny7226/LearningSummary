## Index
[Spring MVC](https://github.com/Danny7226/LearningSummary/blob/main/java/README.md#spring-mvc)

[Java memory model](https://github.com/Danny7226/LearningSummary/blob/main/java/README.md#java-memory-model)

[Weaving](https://github.com/Danny7226/LearningSummary/blob/main/java/README.md#weaving)

[Dynamic proxy JDK](https://github.com/Danny7226/LearningSummary/blob/main/java/README.md#dynamic-proxy-jdk)

[Tomcat, java EE, java servlet, and socket](https://github.com/Danny7226/LearningSummary/blob/main/java/README.md#tomcat-java-ee-java-servlet-and-socket)

[JVM classloader and Spring bean](https://github.com/Danny7226/LearningSummary/tree/main/java#jvm-classloader-and-spring-bean-management)

[Reentrant lock]()

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

### JVM Classloader and Spring Bean management
* TODO read more on Tomcat and how Tomcat defines its classloader
* JVM loads Class dynamically on-demand. When a class is needed, JVM will check ClassLoader to see if a class is loaded (top down, native parent ClassLoader first, then down to custom ClassLoader)
  * If Class is not loaded, JVM loads it
  * If two threads in the same JVM instance concurrently need access to the same class that hasn't been loaded before, the JVM ensures that the class is loaded only once
```agsl
public class MyClassLoader extends ClassLoader {
    // this might introduce dead lock, a refined version is to lock based on an Object with a className
    // https://docs.oracle.com/javase/7/docs/technotes/guides/lang/cl-mt.html -> getClassLoadingLock(name)
    // native class loader might be singleton use an reentrantlock to avoid any deadlock
    private final Object lock = new Object();

    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        synchronized (lock) {
            // Check if the class has already been loaded
            Class<?> loadedClass = findLoadedClass(name);
            if (loadedClass != null) {
                return loadedClass;
            }

            try {
                // Load the class bytes from file or any source
                byte[] classData = loadClassData(name);
                return defineClass(name, classData, 0, classData.length);
            } catch (IOException e) {
                throw new ClassNotFoundException("Class '" + name + "' not found.", e);
            }
        }
    }

    private byte[] loadClassData(String className) throws IOException {
        // For demonstration purposes, load class bytes from a file
        String classFileName = className.replace('.', File.separatorChar) + ".class";
        try (FileInputStream fis = new FileInputStream(classFileName);
             ByteArrayOutputStream bos = new ByteArrayOutputStream()) {

            int data;
            while ((data = fis.read()) != -1) {
                bos.write(data);
            }
            return bos.toByteArray();
        }
    }
    
    @Override
    private Class<T> findLoadedClass(String name) {
        // check loaded classes first, and if not, parent.loadClass(name);
        // if parent.loadClass(name) cannot load, CustomerClassLoader logic will be trigered
        
        // this behavior can be overriden in CustomClassLoader to not invoke parent first 
    }

    public static void main(String[] args) throws ClassNotFoundException {
        // Example usage of the custom class loader
        MyClassLoader myClassLoader = new MyClassLoader();
        Class<?> loadedClass = myClassLoader.loadClass("com.example.MyClass");
        // Use the loaded class as needed
    }
}
```
* example of `getClassLoadingLock(name)`
```agsl
// Emulated getClassLoadingLock method
private Object getClassLoadingLock(String className) {
    synchronized (classLoadingLocks) {
        // Use the class name as a key to retrieve the lock object
        classLoadingLocks.putIfAbsent(className, new Object());
        return classLoadingLocks.get(className);
    }
}
```

  * CustomClassLoader is beneficial in many ways, one of them is when doing AOP
    * CustomerClassLoader can add behaviors to loaded classes
    * You can set CustomClassLoader as the default classloader
      * this will only affect classes loaded after setting this class loader. Classes already loaded before setting this will continue to use the previous class loader.
```agsl
public class MyClass {
    public static void main(String[] args) {
        // Create an instance of your custom class loader
        CustomClassLoader customClassLoader = new CustomClassLoader();

        // Set your custom class loader as the default class loader for the application
        Thread.currentThread().setContextClassLoader(customClassLoader);

        // Now, any classes loaded within this thread will use your custom class loader
        // You'll need to load or run your application logic here
    }
}
```

* Spring Bean has multiple types, Singleton Bean, Session Bean, Prototype Bean, etc
  * Singleton beans are usually constructed during Application Context initialization, and will be maintained through the whole application lifecycle
```agsl
@Component
@Lazy
public class LazyInitializedBean {
    // Implementation
}
```


### Reentrant lock
```agsl
public class CustomReentrantLock {
    private boolean isLocked = false;
    private Thread lockedBy = null;
    private int lockCount = 0;

    public synchronized void lock() throws InterruptedException {
        Thread currentThread = Thread.currentThread();
        while (isLocked && lockedBy != currentThread) {
            // If the lock is held by another thread and it's not reentrant,
            // wait until it's released.
            wait();
        }
        isLocked = true;
        lockedBy = currentThread;
        lockCount++;
    }

    public synchronized void unlock() {
        if (Thread.currentThread() == lockedBy) {
            lockCount--;

            if (lockCount == 0) {
                isLocked = false;
                lockedBy = null;
                notify();
            }
        }
    }
}
```

### AspectJ with annotation
```agsl
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface LogExecutionTime {
    String value() default "Execution Time Logged";
}

@Aspect
public class LoggingAspect {

    @Around("@annotation(logExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint, LogExecutionTime logExecutionTime) throws Throwable {
        long startTime = System.currentTimeMillis();

        Object result;
        try {
            result = joinPoint.proceed();
        } finally {
            long endTime = System.currentTimeMillis();
            long executionTime = endTime - startTime;

            String message = logExecutionTime.value();
            System.out.println(message + ": " + joinPoint.getSignature() + " executed in " + executionTime + "ms");
        }

        return result;
    }
}
```


### Spring Annotation
* Configuration + Bean vs Bean itself
  * `Configuration` indicates CGLib and proxy will be introduced when generating beans inter-dependently within the class
    * IoC container will manager the beans registered within this class and therefore `singleton` will be applied
  * No `Configuration` annotation class will not utilize CGlib proxy and therefore a new instance of bean will be generated everytime it's needed