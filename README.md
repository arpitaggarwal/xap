# XAP

## 1. Service Grid
A service grid is composed of one or more machines (service grid nodes) running a Service Grid Agent (or GSA), and provides a framework to deploy and monitor applications on those machines, in our case the Data Grid.

## 2. Service Grid Components

 - GSM - Grid Service Manager
 - GSC - Grid Service Container
 - LUS - The Lookup Service
 - GSA - Grid Service Agent

### GSM - Grid Service Manager

The Grid Service Manager is the component which manages the deployment and life cycle of the processing unit.

When a processing unit is uploaded to the GSM (using one of GigaSpaces deployment tools: UI, CLI, API), the GSM analyzes the deployment descriptor and determines how many instances of the processing unit should be created, and which containers should host them. It then ships the processing unit code to the relevant containers and instructs them to instantiate the processing unit instances. This phase in the deployment process is called provisioning.

Once provisioned, the GSM continuously monitors the processing unit instances to determine if they’re functioning properly or not. When a certain instance fails, the GSM identifies that and re-provisions the failed instance on to another GSC, thus enforcing the processing unit’s SLA.


### GSC - Grid Service Container

The Grid Service Container provides an isolated runtime for one (or more) processing unit instance, and exposes its state to the GSM.

The GSC can be perceived as a node on the grid, which is controlled by The Grid Service Manager. The GSM provides commands of deployment and un-deployment of the Processing Unit instances into the GSC. The GSC reports its status to the GSM.

### LUS - The Lookup Service

The Lookup Service provides a mechanism for services to discover each other. Each service can query the Lookup service for other services, and register itself in the Lookup Service so other services may find it. For example, the GSM queries the LUS to find active GSCs.

Note that the Lookup service is primarily used for establishing the initial connection - once service X discovers service Y via the Lookup Service, it usually creates a direct connection to it without further involvement of the Lookup Service.

Service registrations in the LUS are lease-based, and each service periodically renews its lease. That way, if a service hangs or disconnects from the LUS, its registration will be cancelled when the lease expires.

The Lookup Service can be configured for either a multicast or unicast environment (default is multicast).

### Grid Service Agent

The Grid Service Agent (GSA) is a process manager that can spawn and manage Service Grid processes (Operating System level processes) such as The Grid Service Manager, The Grid Service Container, and The Lookup Service. Typically, the GSA is started with the hosting machine’s startup. Using the agent, you can bootstrap the entire cluster very easily, and start and stop additional GSCs, GSMs and lookup services at will.

Usually, a single GSA is run per machine. If you’re setting up multiple Service Grids separated by Lookup Groups or Locators, you’ll probably start a GSA per machine per group.

The GSA exposes the ability to start, restart, and kill a process either using the Administration and Monitoring API or the GigaSpaces UI.


## 3. Launch Service Grid
Launch a single node service grid on our machine. To start the service grid, simply run the `gs-agent` script from the product’s bin folder, as follows:

```
./gs-agent.sh
```

## 4. Deploying the Data Grid or Space
```
./gs.sh deploy-space -cluster total_members=2,1 myGrid
```
Above command deploys a Data Grid (aka space) called myGrid with 2 partitions and 1 backup per partition (hence the 2,1 syntax).

## 5. Launch web console
```
./gs-webui.sh
```
Once command command executes open a browser and goto to `http://localhost:8099` and the login screen for the admin application will open up. No username and password needed.

## 6. Launch GigaSpaces Management Center

```
./gs-ui.sh
```

## 7.  Build, Deploy and Undeploy xap-example-web-plain 11.0.0-14800-RELEASE

Move to directory `/gigaspaces-xap-premium-11.0.0-ga/examples/web/plain` and run following commands:

```
./build.sh clean
./build.sh compile
./build.sh package
./build.sh deploy
```

## 8. Deploy 2 Instances of xap-example-web-plain 11.0.0-14800-RELEASE

- Move to directory `/gigaspaces-xap-premium-11.0.0-ga/examples/web/plain/src/main/webapp/META-INF/spring` and modify the `sla.xml` number-of-instances equals to 2, as follows:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:os-sla="http://www.openspaces.org/schema/sla"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
       http://www.openspaces.org/schema/sla http://www.openspaces.org/schema/11.0/sla/openspaces-sla.xsd">

    <!--
    	The SLA determines the number of web container instances that host the application, in our case 1
     -->
    <os-sla:sla number-of-instances="2" max-instances-per-vm="1"/>

</beans>
```

- Move to directory `/gigaspaces-xap-premium-11.0.0-ga/examples/web/plain` and run following commands:

```
./build.sh clean
./build.sh compile
./build.sh package
./build.sh deploy
```

## 9. Configuring Dynamic Load Balancing

- Add below statements in `/etc/apache2/httpd.conf`:

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule proxy_connect_module modules/mod_proxy_connect.so
LoadModule proxy_http_module modules/mod_proxy_http.so

ProxyPass /balancer !

# Proxy Management

<Location /balancer>
SetHandler balancer-manager

Order Deny,Allow
Deny from all
Allow from all
</Location>

ProxyStatus On
<Location /status>
SetHandler server-status

Order Deny,Allow
Deny from all
Allow from all
</Location>

Include /Users/ArpitAggarwal/gigaspaces-xap-premium-11.0.0-ga/tools/apache/gigaspaces/*.conf
```

- Create an Apache GigaSpaces directory so that the Apache load balancing agent can use it as the location of the automatically generated load balancer configurations files it creates, following below commands:

```
cd /Users/ArpitAggarwal/gigaspaces-xap-premium-11.0.0-ga/tools/apache/
mkdir gigaspaces
```

- Start apache-lb-agent.sh moving to directory `/gigaspaces-xap-premium-11.0.0-ga/tools/apache`, as follows:

```
./apache-lb-agent.sh -apachectl /usr/sbin/apachectl -apache /etc/apache2 -conf-dir /Users/ArpitAggarwal/gigaspaces-xap-premium-11.0.0-ga/tools/apache/gigaspaces
```

- Once successfully started, point browser at `http://localhost/balancer`.



Source: http://docs.gigaspaces.com/xap110tut/
