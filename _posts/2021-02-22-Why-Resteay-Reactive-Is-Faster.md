---
layout:     page
title:      "What Makes Resteasy Reactive Faster"
subtitle:   ""
date:       2021-02-26
author:     "Jim Ma"
header-img: "img/post-bg-06.jpg"
---
Since [Quarkus 1.11](https://quarkus.io/blog/quarkus-1-11-0-final-released/), it introduced
Resteasy reactive. It's another jaxrs implementation, but it doesn't have full jaxrs features. 
It is created to tightly integrate with Quarkus and better leverage Qurkus buildtime to do more
annotation and metadata model processing. It is much faster than Resteasy extension. From 
Georgios Andrianakis [presentation of this new feature](https://www.youtube.com/watch?v=WV19zzSCcjk), 
it's 2x more faster than the old jaxrs service backed with Resteasy Quarkus Extension. 
What does improve the performance ?  In this post, I'll try to explain the different parts that Quarkus
Reactive has been improved. 
First, We create a simple jaxrs resource method to 
respond the GET request and returns a simple JSON object for Person object:
```java
import javax.ws.rs.*;
import javax.ws.rs.core.MediaType;
@Path("/reactive")
public class SimpleResource {
    @GET
    @Path("/person")
    @Produces(MediaType.APPLICATION_JSON)
    public Person getPerson() {
        Person person = new Person();
        person.setFirst("Bob");
        person.setLast("Builder");
        return person;
    }
    @POST
    @Path("/person")
    @Produces(MediaType.APPLICATION_JSON)
    @Consumes(MediaType.APPLICATION_JSON)
    public Person getPerson(Person person) {
        if (BlockingOperationControl.isBlockingAllowed()) {
            throw new RuntimeException("should not have dispatched");
        }
        return person;
    }
}
```
The Person is a simple pojo object too and it only contains the first name and last name fields:
```java
public class Person {
    private String first;
    private String last;

    public Person() {
    }

    public String getFirst() {
        return this.first;
    }

    public void setFirst(String first) {
        this.first = first;
    }

    public String getLast() {
        return this.last;
    }

    public void setLast(String last) {
        this.last = last;
    }
}
```
When this jaxrs resource is started, it replies the json response for GET request: 
```
{"first":"Bob","last":"Builder"}
```
Then we use the wrk benchmark tool to compare the result for resteasy-reactive and resteasy extension.
For resteasy-reactive extension, it get the requests/sec number **34250**. 
```
2021-02-23 15:07:16,398 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy-reactive, resteasy-reactive-jackson]
Running 1m test @ http://localhost:8081/reactive/person
  30 threads and 30 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.93ms   20.97ms 479.69ms   99.06%
    Req/Sec     1.16k   618.09     6.19k    73.44%
  2058345 requests in 1.00m, 202.19MB read
Requests/sec:  34250.65
Transfer/sec:      3.36MB
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 69.26 s - in org.acme.resteasy.ReactiveResourceTest
```

Switch to resteasy extension, it only gets 13144 requests/sec. 

```
2021-02-23 15:05:17,409 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy, resteasy-jackson]
Running 1m test @ http://localhost:8081/resteasy/person
  30 threads and 30 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     4.41ms   18.06ms 433.04ms   98.42%
    Req/Sec   443.56    240.21     1.77k    64.75%
  789747 requests in 1.00m, 77.58MB read
Requests/sec:  13144.62
Transfer/sec:      1.29MB
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 68.784 s - in org.acme.resteasy.ResteasyResourceTest
2021-02-23 15:06:18,251 INFO  [io.quarkus] (main) Quarkus stopped in 0.116s
```

What does it lead to this result and resteasy-reactive gets this big performance result? These things could be the 
key success factors.   
### Match resource method with Request URI in build time
For a jaxrs implementation, it always needs to dispatch the request URI to resource method. In JaxRS spec, there is detailed 
algorithm to talk about how to match request with resource method(If you are interested, please look at JSR 399 section 3.7)
resteasy-reactive extension moves this job to build time. This means the resource method invoker will be generated in byte code 
through analyzing the resource method annotation and signature. From the decompiled byte code by resteasy-reactive, 
```java
public class ResteasyReactiveProcessor$setupEndpoints-1591811826 implements StartupTask {
   public void deploy(StartupContext var1) {
      var1.setCurrentBuildStepName("ResteasyReactiveProcessor.setupEndpoints");
      Object[] var2 = new Object[147];
      this.deploy_0(var1, var2);
      this.deploy_1(var1, var2);
   }

   public void deploy_0(StartupContext var1, Object[] var2) {
      ...
      Supplier var18 = ((ResteasyReactiveRecorder)var6).invoker("org.acme.resteasy.SimpleResource$quarkusrestinvoker$getPerson_55543b09225dc5c83bf2e894ae037f590fe976a7");
      var1.putValue("proxykey58", var18);
      ...
      Object var309 = var1.getValue("proxykey58");
      ((ServerResourceMethod)var304).setInvoker((Supplier)var309);
      ((ServerResourceMethod)var304).setHttpMethod("GET"); 
      ...
   }
```

resteasy extension relies on Resteasy core function to match the resouce method with the procedure defined in jaxrs spec. 
Although the matched result will be added to cache and the same request doesn't have to run this match method overtime, it still
consumes the double cpu time compare with resteasy-reactive. 

![resteasy-reactive-resource-method-invoker](/img/qreactive/reactive.png)

This is resteasy-reactive resouce method cpu time. It shows the total resource method invocation time is 
**30975ms**

We use the same parameter to start wrk and run against resteasy extension. The total resource method cpu time is 
:**53236ms**. The resteasy-reactive resource method invocation time is nearly half of resteasy's.

![resteasy-resource-method-invoker](/img/qreactive/resteasy.png)

In addition to resource method invocation time, there is another **43410**ms takes to match the resource method.
This cpu consumption is listed below the resource method invocation. There is still another improvement to make 
resteasy-reactive faster. Let's continue with the second factor.

### Remove the reflection for method invocation
To make method invocation faster, resteasy-reactive generates byte code to call the object directly:
```java
public class SimpleResource$quarkusrestinvoker$getPerson_55543b09225dc5c83bf2e894ae037f590fe976a7 implements EndpointInvoker {
   public Object invoke(Object var1, Object[] var2) {
      return ((SimpleResource)var1).getPerson();
   }
}
```
instead of using the reflection method call like : 

```java
reflectMethod.invoke(targetObject, args);
```

### Match jackson reader/writer once instead of each request
The another job finished in build and start time by resteasy-reactive is get the correct reader/writer to read/write 
message. It is scanning the resource method annotation , analyzing the @Consume @Produce annotation value to 
find the correct component to read/write the message. From the generate byte code, the Jackson reader and writer
alone with other builtin reader and writer will be added to runtime code:
```java
      BeanFactory var35 = ((ResteasyReactiveCommonRecorder)var6).factory("io.quarkus.resteasy.reactive.jackson.runtime.serialisers.JacksonMessageBodyWriter", (BeanContainer)var34);
      var1.putValue("proxykey62", var35);
      ResourceWriter var36 = new ResourceWriter();
      var2[5] = var36;
      Object var37 = var2[5];
      List var38 = Collections.singletonList("application/json");
      ((ResourceWriter)var37).setMediaTypeStrings(var38);
      Object var39 = var1.getValue("proxykey62");
      ((ResourceWriter)var37).setFactory((BeanFactory)var39);
      Boolean var40 = Boolean.valueOf((boolean)0);
      ((ResourceWriter)var37).setBuiltin((Boolean)var40);
``` 
There is a FixedProduceHandler selects this jackson writer during bootstrap and set there to write each response:
![reactive-writer](/img/qreactive/reactive-writer.png)

resteasy extension selects the writer as JAXRS spec defined in section 3.8 in runtime. This still consumes a lot of cpu time as
the following profiler stack shows:

![resteasy-message-writer](/img/qreactive/resteasy-writer.png)

### Running all things in eventloop/unblock thread

Now both resteasy and resteasy-reactive extension all relied on vertx underneath to handle the http request.
Vertx is trying to make all things reactive/noblocking and finish the event/process in 2000ms by default. 
If it doesn't, there is thread blocking checker will warn something like :
```java
2021-02-24 11:38:08,673 WARNING [io.ver.cor.imp.BlockedThreadChecker] (vertx-blocked-thread-checker) Thread Thread[vert.x-eventloop-thread-6,5,main]=Thread[vert.x-eventloop-thread-6,5,main] has been blocked for 508280 ms, time limit is 2000 ms: io.vertx.core.VertxException: Thread blocked
``` 
Due to reasteasy-reactive get all things speed up and it dramatically decreases the time to find the resource method, 
get the correct jackson writer , invoke the resource object without using reflection.It can running all things in an eventloop thread
instead of using a worker thread like what resteasy extension does : 
```java
public class VertxRequestHandler implements Handler<RoutingContext> {
    @Override
    public void handle(RoutingContext request) {
        // have to create input stream here.  Cannot execute in another thread
        // otherwise request handlers may not get set up before request ends
        InputStream is;
        if (request.getBody() != null) {
            is = new ByteArrayInputStream(request.getBody().getBytes());
        } else {
            is = new VertxInputStream(request, readTimeout);
        }
        if (BlockingOperationControl.isBlockingAllowed()) {
            try {
                dispatch(request, is, new VertxBlockingOutput(request.request()));
            } catch (Throwable e) {
                request.fail(e);
            }
        } else {
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        dispatch(request, is, new VertxBlockingOutput(request.request()));
                    } catch (Throwable e) {
                        request.fail(e);
                    }
                }
            });
        }

    }
```
These could be the things get a jaxrs service much faster with resteasy-reactive in quarkus. It might miss 
something or something I don't know. If you find something, please comment to let me know or correct me. 
 



 




 






 


 
  
