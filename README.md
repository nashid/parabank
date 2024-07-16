# Introduction
The ParaBank demo web application and associated web services (SOAP and REST) from Parasoft.

# Build and Install
1. Build the ParaBank application using Maven (`mvn clean install`). After a successful build, deploy the `parabank.war` (located in `/target`) into an Apache Tomcat container.

NOTE: if using the coverage agent when running the functional/manual tests (see below), execute the build using "mvn -Dmaven.test.skip=true clean install jtest:monitor"

* ParaBank uses a built-in HyperSQL database. You must shut down all running instances of **ParaBank** and *HyperSQL* for the build to succeed. Otherwise several tests may fail, since there are a number of ports that are shared between the test instances and the real thing. See [changing default ports](#changing-default-ports).

## Apache Tomcat notes
* Java `17` is recommended. Oracle JDK or Zulu OpenJDK is preferred.

* Install with sdk man:

```
sdk install java 17.0.11.fx-zulu
sdk use java 17.0.11.fx-zulu
```
* 
* Apache Tomcat `9` is recommended.

* To prevent verbose cache warnings in the tomcat log::

 ```text
 Oct 06, 2016 3:53:33 PM org.apache.catalina.webresources.Cache getResource
 WARNING: Unable to add the resource at [/WEB-INF/lib/wss4j-ws-security-stax-2.1.3.jar] to the cache because there was insufficient free space available after evicting expired cache entries - consider increasing the maximum size of the cache
 ```

Add the following to the `<tomcat install>/config/context.xml` make sure to place this block before `</Context>` tag. This setting will provide `100 MB`  for caching:

  ```xml
  <Resources cachingAllowed="true" cacheMaxSize="102400" />
  ```
* To support clean re-deployments of **ParaBank** please add the following to the `<Context>` tag of the `<tomcat install>/config/context.xml`

 ```xml
 <Context antiResourceLocking="true">
 ```

### Stop running processes  
- Verify that the process is terminated:
```
lsof -i :9001
```

## Changing default ports
HyperSQL listens on the default port number `9001`. This port number can conflict with other instances of HyperSQL and may conflict with certain other applications such as the "Intel(R) Graphics Command Center Service". To change the port number to something else like `9002` , modify the following files located under "src/main/resources" or in a deployed WAR under "WEB-INF/classes":

* **applicationContext-hsqldb.xml**: This file configures HyperSQL. Look for the "hsqldb" bean and its "props". Add the following "prop", giving it the desired port number:

```
<prop key="server.port">9002</prop>
```

* **jdbc.properties**: This file configures the JDBC client connection used by ParaBank. Change "jdbc.url" to include the desired port number:

```
jdbc.url=jdbc:hsqldb:hsql://localhost:9002/parabank
```

* **jdbcBookstore.properties**: This file configures the JDBC client connection used by Bookstore. Change the following properties to include the desired port number:

```
jdbc.bookstoreURL=jdbc:hsqldb:hsql://localhost:9002/bookstore
jdbc.bookstorePort=9002
```

Apache ActiveMQ listens on the default port number `61616`. This port number can conflict with other instances of ActiveMQ. To change the default port number to something else like `61617`, modify the following file located under "src/main/resources" or in a deployed WAR under "WEB-INF/classes":

* **applicationContext-jms.xml**: This file configures ActiveMQ. Modify the port number in the "uri" for the transportConnector:

```
<amq:transportConnector uri="tcp://0.0.0.0:61617?transport.daemon=true" />
```

# Test scripts
* All scripts exist in two flavors (.bat and .sh) for Windows and Linux respectively.
* All scripts should be executed from project directory and require the following Parasoft products (and versions)
** Parasoft DTP 5.3.2
** Parasoft Jtest 10.3.2
** Parasoft SOAtest 9.10.1

Script                                 | Description
-------------------------------------- | -----------
__deploy-jtest-monitor(.sh\|.bat)__    | Deploys the Jtest Monitor package (created using mvn goal jtest:monitor) into directory specified by APP_COVERAGE_DIR in `set-vars(.sh|.bat)`
__jtest-ft-cov(.sh\|.bat)__            | Executes Parasoft Jtest DTP Engine for processing monitored coverage during functional testing with SOAtest
__jtest-mt-cov(.sh\|.bat)__            | Uploads results from manual testing efforts (managed using the Parasoft Coverage Agent Manager) and processing coverage data captured during manual testing
__jtest-sa-ut(.sh\|.bat)__             | Executes Parasoft Jtest DTP Engine for Static Analysis and Unit Testing results/coverage
__jtest-sa-ut-delta(.sh\|.bat)__       | Same operation as `jtest-sa-ut(.sh|.bat)` but used to rescan code based for localized changes - used for demonstration purposes when scanning 'dirty' branch
__set-vars(.sh\|.bat)__                | Utility script called by other scripts to consistently set BUILD_ID and the Project name sent to DTP
__soatest(.sh\|.bat)__                 | Executes Parasoft SOAtest API and Web functional tests (including integration with Jtest DTP Engine for monitoring code coverage)

## Setup
set-vars.(.sh\|.bat): setup JTEST_HOME and SOATEST_HOME environment variable before running any script.
all reports will be stored under target/report/<build ID> directory.
on Windows, 7zip must be installed (default to C:\Program Files\7-zip) to run deploy-jtest-monitor.bat script.

# Errors that I have encountered

- After running the application, I encountered the following error:
  http://localhost:8080/parabank

```
HTTP Status 404 – Not Found
Type Status Report

Message The requested resource [/parabank] is not available

Description The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.

Apache Tomcat/9.0.91
```

- When I open the http://localhost:8080/manager and start the `/parabank` application, I encountered the following error:

```
FAIL - Application at context path [/parabank] could not be started
FAIL - Encountered exception [org.apache.catalina.LifecycleException: Failed to start component [org.apache.catalina.webresources.StandardRoot@545a0f4]]
```

- In the `catalina.out`, I get the following error:

```
15-Jul-2024 14:42:06.731 SEVERE [http-nio-8080-exec-1] org.apache.catalina.startup.ExpandWar.copy Error copying [/Users/nashid/repos/apache/apache-tomcat-9.0.91/temp/0-parabank] to [/Users/nashid/repos/apache/apache-tomcat-9.0.91/temp/3-parabank]
	java.io.FileNotFoundException: /Users/nashid/repos/apache/apache-tomcat-9.0.91/temp/0-parabank (No such file or directory)
		at java.base/java.io.FileInputStream.open0(Native Method)
		at java.base/java.io.FileInputStream.open(FileInputStream.java:216)
		at java.base/java.io.FileInputStream.<init>(FileInputStream.java:157)
		at org.apache.catalina.startup.ExpandWar.copy(ExpandWar.java:254)
		at org.apache.catalina.startup.ContextConfig.antiLocking(ContextConfig.java:902)
		at org.apache.catalina.startup.ContextConfig.beforeStart(ContextConfig.java:942)
		at org.apache.catalina.startup.ContextConfig.lifecycleEvent(ContextConfig.java:292)
		at org.apache.catalina.util.LifecycleBase.fireLifecycleEvent(LifecycleBase.java:109)
		at org.apache.catalina.util.LifecycleBase.setStateInternal(LifecycleBase.java:385)
		at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:163)
		at org.apache.catalina.manager.ManagerServlet.start(ManagerServlet.java:1303)
		at org.apache.catalina.manager.HTMLManagerServlet.start(HTMLManagerServlet.java:642)
		at org.apache.catalina.manager.HTMLManagerServlet.doPost(HTMLManagerServlet.java:188)
		at javax.servlet.http.HttpServlet.service(HttpServlet.java:555)
		at javax.servlet.http.HttpServlet.service(HttpServlet.java:623)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:199)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:144)
		at org.apache.catalina.filters.CsrfPreventionFilter.doFilter(CsrfPreventionFilter.java:428)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:168)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:144)
		at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:51)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:168)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:144)
		at org.apache.catalina.filters.HttpHeaderSecurityFilter.doFilter(HttpHeaderSecurityFilter.java:129)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:168)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:144)
		at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:168)
		at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:90)
		at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:597)
		at org.apache.catalina.valves.RequestFilterValve.process(RequestFilterValve.java:355)
		at org.apache.catalina.valves.RemoteAddrValve.invoke(RemoteAddrValve.java:54)
		at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:130)
		at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:93)
		at org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:660)
		at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
		at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:346)
		at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:388)
		at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:63)
		at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:936)
		at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1791)
		at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:52)
		at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1190)
		at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
		at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:63)
		at java.base/java.lang.Thread.run(Thread.java:840)
15-Jul-2024 14:42:08.571 INFO [http-nio-8080-exec-2] org.apache.catalina.util.LifecycleBase.stop The stop() method was called on component [WebappLoader[StandardEngine[Catalina].StandardHost[localhost].StandardContext[/parabank]]] after stop() had already been called. The second call will be ignored.
15-Jul-2024 14:42:08.571 INFO [http-nio-8080-exec-2] org.apache.catalina.util.LifecycleBase.destroy The destroy() method was called on component [org.apache.catalina.webresources.DirResourceSet@5272fe27] after destroy() had already been called. The second call will be ignored.
15-Jul-2024 14:42:08.572 SEVERE [http-nio-8080-exec-2] org.apache.catalina.startup.ExpandWar.copy Error copying [/Users/nashid/repos/apache/apache-tomcat-9.0.91/temp/0-parabank] to [/Users/nashid/repos/apache/apache-tomcat-9.0.91/temp/4-parabank]
	java.io.FileNotFoundException: /Users/nashid/repos/apache/apache-tomcat-9.0.91/temp/0-parabank (No such file or directory)
		at java.base/java.io.FileInputStream.open0(Native Method)
		at java.base/java.io.FileInputStream.open(FileInputStream.java:216)
		at java.base/java.io.FileInputStream.<init>(FileInputStream.java:157)
		at org.apache.catalina.startup.ExpandWar.copy(ExpandWar.java:254)
		at org.apache.catalina.startup.ContextConfig.antiLocking(ContextConfig.java:902)
		at org.apache.catalina.startup.ContextConfig.beforeStart(ContextConfig.java:942)
		at org.apache.catalina.startup.ContextConfig.lifecycleEvent(ContextConfig.java:292)
		at org.apache.catalina.util.LifecycleBase.fireLifecycleEvent(LifecycleBase.java:109)
		at org.apache.catalina.util.LifecycleBase.setStateInternal(LifecycleBase.java:385)
		at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:163)
		at org.apache.catalina.manager.ManagerServlet.start(ManagerServlet.java:1303)
		at org.apache.catalina.manager.HTMLManagerServlet.start(HTMLManagerServlet.java:642)
		at org.apache.catalina.manager.HTMLManagerServlet.doPost(HTMLManagerServlet.java:188)
		at javax.servlet.http.HttpServlet.service(HttpServlet.java:555)
		at javax.servlet.http.HttpServlet.service(HttpServlet.java:623)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:199)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:144)
		at org.apache.catalina.filters.CsrfPreventionFilter.doFilter(CsrfPreventionFilter.java:428)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:168)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:144)
		at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:51)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:168)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:144)
		at org.apache.catalina.filters.HttpHeaderSecurityFilter.doFilter(HttpHeaderSecurityFilter.java:129)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:168)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:144)
		at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:168)
		at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:90)
		at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:597)
		at org.apache.catalina.valves.RequestFilterValve.process(RequestFilterValve.java:355)
		at org.apache.catalina.valves.RemoteAddrValve.invoke(RemoteAddrValve.java:54)
		at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:130)
		at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:93)
		at org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:660)
		at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
		at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:346)
		at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:388)
		at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:63)
		at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:936)
		at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1791)
		at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:52)
		at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1190)
		at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
		at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:63)
		at java.base/java.lang.Thread.run(Thread.java:840)
15-Jul-2024 14:42:34.724 INFO [main] org.apache.catalina.core.StandardServer.await A valid shutdown command was received via the shutdown port. Stopping the Server instance.
15-Jul-2024 14:42:34.724 INFO [main] org.apache.coyote.AbstractProtocol.pause Pausing ProtocolHandler ["http-nio-8080"]
```

Not sure what went wrong.
