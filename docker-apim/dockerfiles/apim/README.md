# Dockerfile for WSO2 API Manager #
The section defines the step-by-step instructions to build the Docker image for WSO2 API Manager 2.1.0.

## Prerequisites

* [Docker](https://www.docker.com/get-docker) v17.09.0 or above


## How to build an image and run
##### 1. Checkout this repository into your local machine using the following git command.
```
git clone https://github.com/wso2/docker-apim.git
```

>The local copy of the `dockerfile/apim` directory will be referred to as `AM_DOCKERFILE_HOME` from this point onwards.

##### 2. Add JDK, WSO2 API Manager distributions and MySQL connector to `<AM_DOCKERFILE_HOME>/files`
- Download [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 
and extract that in `<AM_DOCKERFILE_HOME>/files` folder.
- Download the WSO2 API Manager 2.1.0 distribution (http://wso2.com/api-management/try-it/)
and extract that in `<AM_DOCKERFILE_HOME>/files` folder.
- Once both JDK and WSO2 API Manager distributions are extracted the folder structure should be as follows;

  ```bash
  <AM_DOCKERFILE_HOME>/files/jdk<version>/
  <AM_DOCKERFILE_HOME>/files/wso2am-2.1.0/
  ```
- Download [MySQL Connector/J](https://dev.mysql.com/downloads/connector/j/) v5.1.45 and then copy that to `<AM_DOCKERFILE_HOME>/files` folder. (for your usage 
I already added this connector in the repository. You do not have to perform this step. If you only need to update the connector version, then you can download
new connector and replace the older one)


>Please refer to [WSO2 Update Manager documentation](https://docs.wso2.com/display/ADMIN44x/Updating+WSO2+Products)
in order to obtain latest bug fixes and updates for the product.

##### 3. Data sourse configuration in `<AM_DOCKERFILE_HOME>/files` (This should be done if you are changing the datasources)
- We have to change the `master-datasources.xml` inside the `datasources` folder according to wso2 documentation (https://docs.wso2.com/display/AM210/Installing+and+Configuring+the+Databases). create schemas according to the names in `master-datasources.xml`
 and replace with your database server url in all places called `{your data source url}`. Replace `useName` and `Password` with your credentials.
 
- Same way `registry.xml` and `user-mgt.xml` should be changed according to that same wso2 documentation.

### Handling Java version Known Issue for wso2 Products (https://bugs.openjdk.java.net/browse/JDK-8189789)
- This issue is handle by the `catalina-server.xml` and `Owasp.CsrfGuard.Carbon.xml` configurations in `<AM_DOCKERFILE_HOME>/files/conf/tomcat` and `<AM_DOCKERFILE_HOME>/files/conf security` folders respectively. I have already added the configured `catalina-server.xml` and `Owasp.CsrfGuard.Carbon.xml` files
  in this repository itself, so that you don't have to worry about that.



##### 4. WSO2 API Manager Dockerfile

This git repository provides a Dockerfile for configuring WSO2 API Manager with custom configurations. Currently placeholders for following configurations have been added:

### Configurations:

- Hostname
  
  ````
  repository/conf/carbon.xml
  <!--
    Host name or IP address of the machine hosting this server
    e.g. www.wso2.org, 192.168.1.10
    This is will become part of the End Point Reference of the
    services deployed on this server instance.
  -->
  <HostName>{load-balancer-hostname}</HostName>
  ````

- Management Hostname

  ````
  repository/conf/carbon.xml
  <!--
    Host name to be used for the Carbon management console
    -->
  <MgtHostName>{load-balancer-hostname}</MgtHostName>
  ````

- API Publisher URL

  ````
  repository/conf/api-manager.xml
  <APIPublisher>
     ...
     <URL>https://{load-balancer-hostname}:{load-balancer-https-port}/publisher</URL>
     ...
  </APIPublisher>
  ````

- API Store URL

  ````
  repository/conf/api-manager.xml
  <APIStore>
     ...
     <URL>https://{load-balancer-hostname}:{load-balancer-https-port}/store</URL>
     ...
  </APIStore>
  ````

- API Gateway URL

  ````
  repository/conf/api-manager.xml
  <Environments>
     <Environment type="hybrid" api-console="true">
        ...
        <!-- Endpoint URLs for the APIs hosted in this API gateway.-->
        <GatewayEndpoint>http://{load-balancer-hostname}:{load-balancer-http-port},https://{load-balancer-hostname}:{load-balancer-https-port}</GatewayEndpoint>
     </Environment>
  </Environments>
  ````

### System APIs

This repository has also included following APIs for the below mentioned reasons:

- Publisher API
  
  Exposes API Publisher UI via the API Gateway.

- Store API

  Exposes API Store UI via the API Gateway.

- Registry API 

  Exposes Carbon Registry API via the API Gateway.
  
- Carbon API
	
  Expose API Carbon UI via the API Gateway.

- Echo API

  This echo API can be used for implementing health checks for the API gateway:

  ````bash
  > GET /echo/WSO2 HTTP/1.1
  > Host: 192.168.1.2:8280
  > User-Agent: curl/7.49.1
  > Accept: */*
  >
  < HTTP/1.1 200 OK
  < Host: 192.168.1.2:8280
  < Content-Type: application/json
  < Accept: */*
  < Date: Fri, 28 Oct 2016 04:52:26 GMT
  < Server: WSO2-PassThrough-HTTP
  < Transfer-Encoding: chunked
  <
  * Connection #0 to host 192.168.1.2 left intact
  {"response":{"hello":"WSO2","server-ip":"192.168.1.2"}}
  ````

##### 5. Build the Docker image.
- Navigate to `<AM_DOCKERFILE_HOME>` directory. <br>
  Execute `docker build` command as shown below.
    + `docker build -t wso2am:2.1.0 .`
	

### We need only upto this section for the ecs deployment process.###
    
##### 6. Running the Docker image.
- `docker run -it -p 9443:9443 wso2am:2.1.0`

##### 7. Accessing management console.
- To access the management console, use the docker host IP and port 9443.
    + `https:<DOCKER_HOST>:9443/carbon`
    
>In here, <DOCKER_HOST> refers to hostname or IP of the host machine on top of which containers are spawned.


## How to update configurations
Configurations would lie on the Docker host machine and they can be volume mounted to the container. <br>
As an example, steps required to change the port offset using `carbon.xml` is as follows.

##### 1. Stop the API Manager container if it's already running.
In WSO2 API Manager 2.1.0 product distribution, `carbon.xml` configuration file <br>
can be found at `<DISTRIBUTION_HOME>/repository/conf`. Copy the file to some suitable location of the host machine, <br>
referred to as `<SOURCE_CONFIGS>/carbon.xml` and change the offset value under ports to 1.

##### 2. Grant read permission to `other` users for `<SOURCE_CONFIGS>/carbon.xml`
```
chmod o+r <SOURCE_CONFIGS>/carbon.xml
```

##### 3. Run the image by mounting the file to container as follows.
```
docker run \
-p 9444:9444 \
--volume <SOURCE_CONFIGS>/carbon.xml:<TARGET_CONFIGS>/carbon.xml \
wso2am:2.1.0
```

>In here, <TARGET_CONFIGS> refers to /home/wso2carbon/wso2am-2.1.0/repository/conf folder of the container.


## Docker command usage references

* [Docker build command reference](https://docs.docker.com/engine/reference/commandline/build/)
* [Docker run command reference](https://docs.docker.com/engine/reference/run/)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
