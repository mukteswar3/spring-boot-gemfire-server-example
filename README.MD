# Spring Boot GemFire Server Example

The *Spring Boot GemFire Server Example* project demonstrates how to configure and bootstrap an embedded 
*Pivotal GemFire* or *Apache Geode* Server (i.e. peer cache member) in a *Spring Boot* application JVM process.

NOTE: From this point forward, you can safely assume that *Pivotal GemFire* and *Apache Geode* are interchangeable.
So, when I say *Pivotal GemFire*, I also mean the following documentation applies to *Apache Geode* as well.

This *GemFire Server* is configured using *Spring (Data GemFire)*, Java-based configuration meta-data and assumes (OOTB) 
 the user wants to connect this server to an existing *GemFire Cluster*.  The supposedly, existing *GemFire Cluster* 
is defined by a *GemFire Locator* running on `localhost`, listening on port `10334` (the default *Locator* port)
for member join requests.

The *Spring* Java-based configuration meta-data allows for certain application properties to be configured in order to 
adjust, among other things, *GemFire's* `log-level` or the `cluster` to which this server will connect.

More information on how to run this example is detailed below.

### Requirements

* *Java Development Kit* (JDK) 1.8.0+ (e.g. 1.8.0_72).
* *Gradle* 2.2+

### Dependencies

This project primarily depends on *Spring Boot* `1.3.5.RELEASE`, *Spring Data GemFire* `1.8.2.RELEASE` 
and *Pivotal GemFire* `8.2.0` (although *Spring Data Geode* `1.0.0.APACHE-GEODE-INCUBATING-M2` along with 
*Apache Geode* `1.0.0-incubating.M2` can be substituted in place of *Spring Data GemFire* and *Pivotal GemFire*).
 
You can download distributions of the *JDK* [here](http://www.oracle.com/technetwork/java/javase/downloads/index.html).

This project uses the [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) so no explicit 
*Gradle* installation is necessary.

### Build

To build the *spring-boot-gemfire-server-example* project, run the `gradlew` shell script at the command-line

```
$ gradlew clean build install
```

### Run

To run this example, you must first start a `GemFire Locator` using *Gfsh* (the *GemFire Shell* tool).

For this example, I downloaded, installed and used *Pivotal GemFire* `8.2.0` from the 
[Pivotal Network](https://network.pivotal.io/products/pivotal-gemfire). 

See the *Pivotal GemFire* installation instructions [here](http://gemfire.docs.pivotal.io/docs-gemfire/latest/getting_started/installation/install_intro.html).

Similarly, *Apache Geode* installation instructions can be found [here](http://geode.docs.pivotal.io/docs/prereq_and_install.html).


```
$ echo $GEMFIRE
/Users/jblum/Downloads/Pivotal/GemStone/Products/GemFire/Pivotal_GemFire_820_b17919_Linux

$ gfsh

    _________________________     __
   / _____/ ______/ ______/ /____/ /
  / /  __/ /___  /_____  / _____  / 
 / /__/ / ____/  _____/ / /    / /  
/______/_/      /______/_/    /_/    v8.2.0

Monitor and Manage GemFire


gfsh>start locator --name=GemFireLocator --log-level=config
Starting a GemFire Locator in /Users/jblum/pivdev/lab/GemFireLocator...
.............................
Locator in /Users/jblum/pivdev/lab/GemFireLocator on 10.99.199.9[10334] as GemFireLocator is currently online.
Process ID: 81551
Uptime: 15 seconds
GemFire Version: 8.2.0
Java Version: 1.8.0_72
Log File: /Users/jblum/pivdev/lab/GemFireLocator/GemFireLocator.log
JVM Arguments: -Dgemfire.enable-cluster-configuration=true -Dgemfire.load-cluster-configuration-from-dir=false -Dgemfire.log-level=config -Dgemfire.launcher.registerSignalHandlers=true -Djava.awt.headless=true -Dsun.rmi.dgc.server.gcInterval=9223372036854775806
Class-Path: /Users/jblum/Downloads/Pivotal/GemStone/Products/GemFire/Pivotal_GemFire_820_b17919_Linux/lib/gemfire.jar:/Users/jblum/Downloads/Pivotal/GemStone/Products/GemFire/Pivotal_GemFire_820_b17919_Linux/lib/locator-dependencies.jar

Successfully connected to: [host=10.99.199.9, port=1099]

Cluster configuration service is up and running.

```

Once the *GemFire Locator* (also a *GemFire Manager*) is started, you can use the `list members` 
and `describe member` commands in *Gfsh* to find out more details about the cluster and it's members...
  
```
gfsh>list members
Member Count : 1
Coordinator  : GemFireLocator (jblum-mbpro-2(GemFireLocator:81551:locator)<v0>:28961)

     Name      | Id
-------------- | -----------------------------------------------------
GemFireLocator | jblum-mbpro-2(GemFireLocator:81551:locator)<v0>:28961


gfsh>describe member --name=GemFireLocator
Name        : GemFireLocator
Id          : jblum-mbpro-2(GemFireLocator:81551:locator)<v0>:28961
Host        : 10.99.199.9
Regions     : 
PID         : 81551
Groups      : 
Used Heap   : 60M
Max Heap    : 3641M
Working Dir : /Users/jblum/pivdev/lab/GemFireLocator
Log file    : /Users/jblum/pivdev/lab/GemFireLocator/GemFireLocator.log
Locators    : 10.99.199.9[10334]```
```

You can continue to start and add additional *GemFire Locators* or *GemFire Servers* to the cluster.

However, you must be careful to configure network ports accordingly, especially if you configure and start multiple
*Locators*, *Servers* (especially severs that will also function and server as a *CacheServer* listening for 
cache client connections) or even other *Managers*, since all newly added members will attempt to bind to the 
default port for the aforementioned services respectively (Location, Management and Cache Serving), which will 
lead to `BindExceptions` with ports already in use if not configured properly.
  
At this point, you may start the *Spring Boot GemFire Server* application.  If you changed the *GemFire Locator's* port 
on startup to anything other than the default port (`10334`), then you must specify the *Locator`s* port when starting 
the *Spring Boot GemFire Server* in order for the server to find the *Locator* and join to the cluster
(started and formed in *Gfsh*).

For instance, if you started the *GemFire Locator* like so...

```
gfsh>start locator --name=Example --port=11235
```

Then you must also specify the appropriate [application property](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/resources/application.properties#L4) 
when starting the *Spring Boot GemFire Server* application.

Once configured accordingly and properly, then the *Spring Boot GemFire Server* will be in a runnable state and should
be able to connect to the cluster.  You may run the applicaton from inside your IDE of choice, from the command-line 
using either the *Spring Boot Gradle Plugin*...

```
$gradlew bootRun
```

Or, using the "executable" JAR file (`build/lib/spring-boot-gemfire-server-example-1.0.0.SNAPSHOT.jar`) created during 
the *Gradle* build...

```
$java -jar build/libs/spring-boot-gemfire-server-example-1.0.0.SNAPSHOT.jar
```

This nice thing about the later approach is that you may adjust any of the [application.properties](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/resources/application.properties)
by using Java System properties from the command-line (for instance, adjusting the `gemfire.log-level`)...

```
$java -Dspring.gemfire.log-level=warning -jar build/libs/spring-boot-gemfire-server-example-1.0.0.SNAPSHOT.jar
```

Once the *Spring Boot GemFire Server* successfully starts and connects to the cluster defined by our *Gfsh* started
*Locator*, then it will be possible to interact with this server using *Gfsh*...

```
gfsh>list members
Member Count : 2
Coordinator  : GemFireLocator (jblum-mbpro-2(GemFireLocator:81551:locator)<v0>:28961)

         Name           | Id
----------------------- | ------------------------------------------------------
SpringBootGemFireServer | jblum-mbpro-2(SpringBootGemFireServer:81563)<v1>:21208
GemFireLocator          | jblum-mbpro-2(GemFireLocator:81551:locator)<v0>:28961


gfsh>describe member --name=SpringBootGemFireServer
Name        : SpringBootGemFireServer
Id          : jblum-mbpro-2(SpringBootGemFireServer:81563)<v1>:21208
Host        : 10.99.199.9
Regions     : Factorials
PID         : 81563
Groups      : 
Used Heap   : 60M
Max Heap    : 3641M
Working Dir : /Users/jblum/pivdev/spring-data-examples-workspace/spring-boot-gemfire-server-example
Log file    : /Users/jblum/pivdev/spring-data-examples-workspace/spring-boot-gemfire-server-example
Locators    : localhost[10334]

Cache Server Information
Server Bind              : localhost
Server Port              : 40404
Running                  : true
Client Connections       : 0
```

Note the details of interest:

1. First, you can see here our *Spring Boot GemFire Server* is part of the cluster defined by our *Locator* 
in the `list members` command.
1. Next, our *Spring Boot*-based *GemFire Server* was specifically named "`SpringBootGemFireServer`"; see [here](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/java/org/example/SpringBootGemFireServer.java#L55-L57)
and [here](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/java/org/example/SpringBootGemFireServer.java#L70)).
3. The *Spring* configuration also [defined and started](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/java/org/example/SpringBootGemFireServer.java#L95-L111)
a `CacheServer` (service) on `localhost`, listening on the default port, `40404` (which is [configurable](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/resources/application.properties#L1-L3)).
4. Finally, you will notice the "`Factorials`", `PARTITION` *Region* was [defined](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/java/org/example/SpringBootGemFireServer.java#L113-L126).

For more details on the "`Factorials`" *Region*, run `list regions` and `describe region` in *Gfsh*...

```
gfsh>list regions
List of regions
---------------
Factorials


gfsh>describe region --name=/Factorials
..........................................................
Name            : Factorials
Data Policy     : partition
Hosting Members : SpringBootGemFireServer

Non-Default Attributes Shared By Hosting Members  

 Type  | Name | Value
------ | ---- | -----
Region | size | 0


```

To make the example more useful and interactive, the "`Factorials`" *Region* is exactly as the name implies, it is
a `Map` data structure (`PARTITION` *Region*) containing [factorials](https://en.wikipedia.org/wiki/Factorial).

The keys are numbers for which the *factorial* is computed and the value is the result of the *factorial* computation 
of that number (i.e. key), where *factorial* is defined as `factorial(n) = n!`
(e.g. `factorial(4) = 4 * 3 * 2 * 1 = 24`).

The "`Factorials`" *Region* is not pre-loaded.  Instead, I have [defined](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/java/org/example/SpringBootGemFireServer.java#L139-L165)
a simple *GemFire* `CacheLoader` that computes the *factorial* of a key (number), which was [added](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/java/org/example/SpringBootGemFireServer.java#L121)
to the "`Factorials`" *Region* in the bean definition.

So, in order to compute the *factorial* of (just about) any number, you just need to request a key...

```
fsh>get --region=/Factorials --key=1 --key-class=java.lang.Long
Result      : true
Key Class   : java.lang.Long
Key         : 1
Value Class : java.lang.Long
Value       : 1


gfsh>get --region=/Factorials --key=2 --key-class=java.lang.Long
Result      : true
Key Class   : java.lang.Long
Key         : 2
Value Class : java.lang.Long
Value       : 2


gfsh>get --region=/Factorials --key=3 --key-class=java.lang.Long
Result      : true
Key Class   : java.lang.Long
Key         : 3
Value Class : java.lang.Long
Value       : 6


gfsh>get --region=/Factorials --key=4 --key-class=java.lang.Long
Result      : true
Key Class   : java.lang.Long
Key         : 4
Value Class : java.lang.Long
Value       : 24


gfsh>get --region=/Factorials --key=5 --key-class=java.lang.Long
Result      : true
Key Class   : java.lang.Long
Key         : 5
Value Class : java.lang.Long
Value       : 120


gfsh>get --region=/Factorials --key=6 --key-class=java.lang.Long
Result      : true
Key Class   : java.lang.Long
Key         : 6
Value Class : java.lang.Long
Value       : 720


gfsh>get --region=/Factorials --key=4 --key-class=java.lang.Long
Result      : true
Key Class   : java.lang.Long
Key         : 4
Value Class : java.lang.Long
Value       : 24


gfsh>describe region --name=/Factorials
..........................................................
Name            : Factorials
Data Policy     : partition
Hosting Members : SpringBootGemFireServer

Non-Default Attributes Shared By Hosting Members  

 Type  | Name | Value
------ | ---- | -----
Region | size | 6

```

As you can see, the "`Factorials`" *Region* size changes as we request different keys (numbers).  It is also a 
simple matter to adjust the *Spring* configuration and perhaps add a eviction or expiration policies (along with 
other settings) to prevent this *Region* from growing without bound.


### Running with Apache Geode

As previously alluded to, it is a verfy simple matter to run this example with *Apache Geode* instead of 
*Pivotal GemFire*.  You simply only need to adjust the [dependency](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/build.gradle#L32) 
from `spring-data-gemfire` to `spring-data-geode`.

Additionally, you must change the version to use `$springDataGeodeVersion` from the [gradle.properties](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/gradle.properties#L8) 
file.

However, keep in mind, you must run a *Geode Locator* (instead of a *GemFire Locator*) when attempting to connect
the server since the underlying membership protocol and stack has changed in *Apache Geode* 
(which is using a newer version of JGroups with a completely re-architected *GemFire/Geode Membership Service* (GMS)).


### Running with an Embedded GemFire/Geode Locator

While it is apparent you can form an external cluster and have our *Spring Boot GemFire Server* connect to this 
existing cluster by defining the `gemfire.locators` System property ([here](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/java/org/example/SpringBootGemFireServer.java#L73)),
it is also possible to start this example from a *Spring* perspective.  This is particularly useful in a microservices
-based application architecture hosted in a cloud environment (e.g. *Pivotal CloudFoundry*, which is even possible 
with or without the *PCF GemFire* tile).

This is accomplished by running an "embedded" *GemFire Locator* (service) using the `start-gemfire` *GemFire* System property
([for example](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/java/org/example/SpringBootGemFireServer.java#L78-L80))
in our *Spring Boot GemFire Server* application.
 
Additionally, it is also possible for our *Spring Boot GemFire Server* to become a *GemFire Manager* 
(see [here](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/java/org/example/SpringBootGemFireServer.java#L74-L76),
and specifically the [jmx-manager-start](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/java/org/example/SpringBootGemFireServer.java#L76)
*GemFire* System property, which is configured [here](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/resources/application.properties#L7)).
but also it is possible to run an 

With this *GemFire* System property defined, when our *GemFire* member is started, it is possible to connect to 
this member by using either the `--locator` or `--jmx-manager` command switch to the *Gfsh* `connect` command, 
without any prior existing cluster or members.

Furthermore, once the *Spring Boot GemFire Server* is configured with `start-locator` and started, it is very easy to 
add additional members in *Gfsh* by pointing members to the embedded *Locator* of our *Spring Boot GemFire Server*.

Before proceeding to the following example, please stop the previous cluster...

```
gfsh>stop locator --name=GemFireLocator
Stopping Locator running in /Users/jblum/pivdev/lab/GemFireLocator on 10.99.199.9[10334] as GemFireLocator...
Process ID: 81551
Log File: /Users/jblum/pivdev/lab/GemFireLocator/GemFireLocator.log
...

No longer connected to 10.99.199.9[1099].

No longer connected to 10.99.199.9[1099].


gfsh>list members
"list members" is not available. Reason: Requires connection.
```

Onwards...

#### Example

1. First, I set the `start-locator` *application.properties* [property](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/src/main/resources/application.properties#L8)
accordingly...

```
spring.gemfire.start-locator=localhost[10334]
```

2. Then, I start my *Spring Boot GemFire Server*...

```
$ gradlew bootRun
```

3. Next, I can connect to this server...

```
gfsh>connect --locator=localhost[10334]
Connecting to Locator at [host=localhost, port=10334] ..
Connecting to Manager at [host=10.99.199.9, port=1199] ..
Successfully connected to: [host=10.99.199.9, port=1199]


gfsh>list members
         Name           | Id
----------------------- | ---------------------------------------------------
SpringBootGemFireServer | 10.99.199.9(SpringBootGemFireServer:83057)<v0>:6632


gfsh>disconnect
Disconnecting from: 10.99.199.9[1199]
Disconnected from : 10.99.199.9[1199]


gfsh>connect --jmx-manager=localhost[1199]
Connecting to Manager at [host=localhost, port=1199] ..
Successfully connected to: [host=localhost, port=1199]


gfsh>list members
         Name           | Id
----------------------- | ---------------------------------------------------
SpringBootGemFireServer | 10.99.199.9(SpringBootGemFireServer:83057)<v0>:6632


gfsh>disconnect
Disconnecting from: localhost[1199]
Disconnected from : localhost[1199]


gfsh>connect
Connecting to Locator at [host=localhost, port=10334] ..
Connecting to Manager at [host=10.99.199.9, port=1199] ..
Successfully connected to: [host=10.99.199.9, port=1199]


gfsh>list members
         Name           | Id
----------------------- | ---------------------------------------------------
SpringBootGemFireServer | 10.99.199.9(SpringBootGemFireServer:83057)<v0>:6632

gfsh>
```

As you can see, I can connect 3 different ways:

1. `connect --locator=localhost[10334]`
2. `connect --jmx-manager=localhost[1199]`
3. `connect`

The last option (`connect`) just attempts to connect to a *Locator* on the default host and port 
(i.e. `localhost[10334]`).

When Gfsh connects via a *Locator* (indirectly to the *Manager*) a request is made by the *Locator* to "locate" any
existing *Manager* in the cluster.  If one is found, then the *Locator* returns the coordinates of this *Manager*
to the JMX client (e.g. *Gfsh*).  If a existing *Manager* is not found, the *Locator* becomes a *Manager*.  Either way,
*Gfsh*, upon receiving the *Manager's* coordinates, will open a JMX (RMI) connection to the *Manager* directly
and therefore bypass the *Locator* on all subsequent requests (i.e. these are JMX/RMI protocol requests; not *Locator*
requests which uses a separate and distinct protocol).  This is why option 2 is the most direct route, when, of course,
the *Manager* coordinates are known in advance.

Now, you can proceed to add additional members (*Locators* and *Servers*, or other members serving as a *Manager*)...

```
gfsh>start locator --name=ExampleLocator --port=11235 --log-level=config
Starting a GemFire Locator in /Users/jblum/pivdev/lab/ExampleLocator...
.......
Locator in /Users/jblum/pivdev/lab/ExampleLocator on 10.99.199.9[11235] as ExampleLocator is currently online.
Process ID: 83108
Uptime: 3 seconds
GemFire Version: 8.2.0
Java Version: 1.8.0_72
Log File: /Users/jblum/pivdev/lab/ExampleLocator/ExampleLocator.log
JVM Arguments: -Dgemfire.default.locators=10.99.199.9:127.0.0.1[10334] -Dgemfire.enable-cluster-configuration=true -Dgemfire.load-cluster-configuration-from-dir=false -Dgemfire.log-level=config -Dgemfire.launcher.registerSignalHandlers=true -Djava.awt.headless=true -Dsun.rmi.dgc.server.gcInterval=9223372036854775806
Class-Path: /Users/jblum/Downloads/Pivotal/GemStone/Products/GemFire/Pivotal_GemFire_820_b17919_Linux/lib/gemfire.jar:/Users/jblum/Downloads/Pivotal/GemStone/Products/GemFire/Pivotal_GemFire_820_b17919_Linux/lib/locator-dependencies.jar

Cluster configuration service is up and running.


gfsh>list members
         Name           | Id
----------------------- | ---------------------------------------------------
SpringBootGemFireServer | 10.99.199.9(SpringBootGemFireServer:83057)<v0>:6632
ExampleLocator          | 10.99.199.9(ExampleLocator:83108:locator)<v1>:34644


gfsh>start server --name=ExampleServer --server-port=12480 --log-level=config
Starting a GemFire Server in /Users/jblum/pivdev/lab/ExampleServer...
....
Server in /Users/jblum/pivdev/lab/ExampleServer on 10.99.199.9[12480] as ExampleServer is currently online.
Process ID: 83117
Uptime: 2 seconds
GemFire Version: 8.2.0
Java Version: 1.8.0_72
Log File: /Users/jblum/pivdev/lab/ExampleServer/ExampleServer.log
JVM Arguments: -Dgemfire.default.locators=10.99.199.9[11235],10.99.199.9:127.0.0.1[10334] -Dgemfire.log-level=config -Dgemfire.use-cluster-configuration=true -XX:OnOutOfMemoryError=kill -KILL %p -Dgemfire.launcher.registerSignalHandlers=true -Djava.awt.headless=true -Dsun.rmi.dgc.server.gcInterval=9223372036854775806
Class-Path: /Users/jblum/Downloads/Pivotal/GemStone/Products/GemFire/Pivotal_GemFire_820_b17919_Linux/lib/gemfire.jar:/Users/jblum/Downloads/Pivotal/GemStone/Products/GemFire/Pivotal_GemFire_820_b17919_Linux/lib/server-dependencies.jar


gfsh>list members
         Name           | Id
----------------------- | ---------------------------------------------------
SpringBootGemFireServer | 10.99.199.9(SpringBootGemFireServer:83057)<v0>:6632
ExampleLocator          | 10.99.199.9(ExampleLocator:83108:locator)<v1>:34644
ExampleServer           | 10.99.199.9(ExampleServer:83117)<v2>:62276

```

Additionally, I can stop my "ExampleLocator" *Locator* member and the cluster is still there.

```
gfsh>stop locator --name=ExampleLocator
Stopping Locator running in /Users/jblum/pivdev/lab/ExampleLocator on 10.99.199.9[11235] as ExampleLocator...
Process ID: 83108
Log File: /Users/jblum/pivdev/lab/ExampleLocator/ExampleLocator.log
...


gfsh>list members
         Name           | Id
----------------------- | ---------------------------------------------------
SpringBootGemFireServer | 10.99.199.9(SpringBootGemFireServer:83057)<v0>:6632
ExampleServer           | 10.99.199.9(ExampleServer:83117)<v2>:62276

```

This is because the cluster is defined by the "embedded" *Locator* running in our *Spring Boot GemFire Server*.


### Additional Notes

The only thing you cannot do with a *Spring*-based (configured and embedded) *GemFire Server* JVM process is to run
`status server` or `stop server` in *Gfsh* (whether with *Spring Boot* or otherwise) on the member.

The main reason for this is because the *Spring (Boot) GemFire Server* node was not started using *GemFire*
infrastructure classes (namely, `ServerLauncher`) which registers a *Locator/Server* node with the *GemFire*
management infrastructure to be "controlled" (managed) accordingly.

However, any *Spring*-based node is visible from *Gfsh* and other tools (e.g. *Pulse*, even *JConsole* or *JVisualVM*)
and can be interacted with accordingly.

### Conclusion

There you have it.  Clearly there are many different ways to start and form a *Pivotal GemFire* or *Apache Geode* 
cluster with and without *Spring*.  You are only limited by your imagination.

**Magic!**


### Issues

For any issues or feedback on this project, please file a [GitHub Issue](https://github.com/jxblum/spring-boot-gemfire-server-example/issues).  


### Known Issues

The only known issue (at the moment) is when running the *Spring Boot GemFire Server* using the "executable" JAR file,
*Pivotal GemFire* (and possibly *Apache Geode*; not sure if this was fixed in *Geode* already) will log an 
`IllegalStateException` stating that *GemFire/Geode* cannot find the *Gfsh* command classes on the classpath.

For instance...

```
[warn 2016/06/21 16:10:00.482 PDT <main> tid=0xb] (tid=11 msgId=0) Required Commands classes were not loaded. Check logs for errors.
java.lang.IllegalStateException: Required Commands classes were not loaded. Check logs for errors.
	at com.gemstone.gemfire.management.internal.cli.CommandManager.raiseExceptionIfEmpty(CommandManager.java:240)
	at com.gemstone.gemfire.management.internal.cli.CommandManager.loadCommands(CommandManager.java:179)
	at com.gemstone.gemfire.management.internal.cli.CommandManager.<init>(CommandManager.java:77)
	at com.gemstone.gemfire.management.internal.cli.CommandManager.getInstance(CommandManager.java:269)
	at com.gemstone.gemfire.management.internal.cli.CommandManager.getInstance(CommandManager.java:249)
	at com.gemstone.gemfire.management.internal.cli.remote.CommandProcessor.<init>(CommandProcessor.java:50)
	at com.gemstone.gemfire.management.internal.cli.remote.MemberCommandService.<init>(MemberCommandService.java:32)
	at com.gemstone.gemfire.management.cli.CommandService.createLocalCommandService(CommandService.java:138)
	at com.gemstone.gemfire.management.internal.beans.MemberMBeanBridge.<init>(MemberMBeanBridge.java:352)
	at com.gemstone.gemfire.management.internal.beans.ManagementAdapter.handleCacheCreation(ManagementAdapter.java:152)
	at com.gemstone.gemfire.management.internal.beans.ManagementListener.handleEvent(ManagementListener.java:111)
	at com.gemstone.gemfire.distributed.internal.InternalDistributedSystem.notifyResourceEventListeners(InternalDistributedSystem.java:2243)
	at com.gemstone.gemfire.distributed.internal.InternalDistributedSystem.handleResourceEvent(InternalDistributedSystem.java:503)
	at com.gemstone.gemfire.internal.cache.GemFireCacheImpl.init(GemFireCacheImpl.java:1009)
	at com.gemstone.gemfire.internal.cache.GemFireCacheImpl.create(GemFireCacheImpl.java:682)
	at com.gemstone.gemfire.cache.CacheFactory.create(CacheFactory.java:182)
	at com.gemstone.gemfire.cache.CacheFactory.create(CacheFactory.java:229)
	at org.springframework.data.gemfire.CacheFactoryBean.createCache(CacheFactoryBean.java:356)
	at org.springframework.data.gemfire.CacheFactoryBean.resolveCache(CacheFactoryBean.java:250)
	at org.springframework.data.gemfire.CacheFactoryBean.init(CacheFactoryBean.java:191)
	at org.springframework.data.gemfire.CacheFactoryBean.getObject(CacheFactoryBean.java:178)
	at org.springframework.data.gemfire.CacheFactoryBean.getObject(CacheFactoryBean.java:88)
	at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.doGetObjectFromFactoryBean(FactoryBeanRegistrySupport.java:168)
	at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.getObjectFromFactoryBean(FactoryBeanRegistrySupport.java:103)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getObjectForBeanInstance(AbstractBeanFactory.java:1590)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:254)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.findAutowireCandidates(DefaultListableBeanFactory.java:1192)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1116)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1014)
	at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:813)
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:741)
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:464)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1123)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1018)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:510)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:482)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:753)
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:839)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:538)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:766)
	at org.springframework.boot.SpringApplication.createAndRefreshContext(SpringApplication.java:361)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:307)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1191)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1180)
	at org.example.SpringBootGemFireServer.main(SpringBootGemFireServer.java:52)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:54)
	at java.lang.Thread.run(Thread.java:745)

[main] INFO org.springframework.data.gemfire.CacheFactoryBean - Connected to Distributed System [SpringBootGemFireServer] as Member [10.99.199.9(SpringBootGemFireServer:82732)<v49>:59271]in Group(s) [[]] with Role(s) [[]] on Host [10.99.199.9] having PID [82732].
```

This is both unfortunate and misleading since as the last log statement indicates, the member starts up 
and joins the cluster just fine.

The reason this `IllegalStateException` occurs is, *GemFire/Geode* contains a bug whereby the member (e.g. server) 
attempts to locate and load the *Gfsh* command classes even when the member is not configured as a *Manager*.

When the member is a *Manager*, the *Locator/Server* needs these command classes to carry out the execution of the 
"remote" commands when issued by a JMX client (e.g. *Gfsh*) to the management service.  Therefore loading them is 
legitimate, but only when the member is actually a *Manager*.

However, equally unfortunate, is that *GemFire/Geode* uses a broken class loading mechanism with it's own "custom",
Java `ClassLoader`.  This is problematic when executing *GemFire/Geode* from an "executable" *Spring Boot* created
JAR file (a "fat" JAR that contains all the dependencies).

Spring Boot also uses a "custom" `ClassLoader` of its own to load dependency JAR files contained in the application JAR 
file itself.  However, *GemFire/Geode* knows nothing about the *Spring Boot* `ClassLoader`, and even though the 
`spring-shell-1.0.0.RELEASE.jar` is contained in the `lib/` sub-directory relative to the root of "executable" JAR file 
(`spring-boot-gemfire-server-example-1.0.0.SNAPSHOT.jar`), it is not able to find and resolve
the *Spring Shell* classes needed by the *Gfsh* command classes running the server (a *Manager*), and so this
`IllegalStateException` occurs when running with the "executable" application JAR file.

This problem does **not** occur when using the *Spring Boot Gradle Plugin* because the application lib dependencies
are not contained in a JAR file and instead, the application is just run from the command-line as if doing...

```
// classpath with spring-shell-1.0.0.RELEASE.jar file

$ java ... -classpath=".." org.example.SpringBootGemFireServer ...
```

I.e. the *Spring Boot Gradle Plugin* just uses the [classpath](https://github.com/jxblum/spring-boot-gemfire-server-example/blob/master/build.gradle#L28-L37)
defined by *Gradle* as declared in `build.gradle` under the `dependencies` closure. 


### Contributions

I welcome any contributions through [Pull Requests](https://github.com/jxblum/spring-boot-gemfire-server-example/pulls).
If you have an idea or improvement, please share.  No CLA necessary.

**Thanks**
