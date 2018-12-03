# Dockerfiles for WSO2 Enterprise Integrator #
This section defines dockerfiles and step-by-step instructions to build docker images for Integrator profiles <br>
provided by WSO2 Enterprise Integrator 6.1.1 <br>

## Prerequisites
* [Docker](https://www.docker.com/get-docker) v17.09.0 or above

## How to build an image and run
##### 1. Checkout this repository into your local machine using the following git command.
```
git clone https://github.com/wso2/docker-ei.git
```

>The local copy of the `dockerfiles` directory will be referred to as `DOCKERFILE_HOME` from this point onwards.

##### 2. Add JDK, WSO2 Enterprise Integrator distribution and required libraries
- Download [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) and
extract into `<DOCKERFILE_HOME>/base/files`.
- Download [WSO2 Enterprise Integrator 6.1.1 distribution](https://wso2.com/integration) and 
extract into `<DOCKERFILE_HOME>/base/files`.
- Once both JDK and WSO2 Enterprise Integrator distribution is extracted, it should be as follows:
```
<DOCKERFILE_HOME>/base/files/jdk<version>
<DOCKERFILE_HOME>/base/files/wso2ei-6.1.1
```
- Download [MySQL Connector/J](https://dev.mysql.com/downloads/connector/j/) v5.1.45 and then copy that to `<DOCKERFILE_HOME>/base/files` folder. (for your usage 
I already added this connector in the repository. You do not have to perform this step. If you only need to update the connector version, then you can download
new connector and replace the older one)


### Handling Java version Known Issue for wso2 Products (https://bugs.openjdk.java.net/browse/JDK-8189789)
- This issue is handle by the `catalina-server.xml` and `Owasp.CsrfGuard.Carbon.xml` configuration files in the `<DOCKERFILE_HOME>/base/files/`. I have already added the configured `catalina-server.xml` and `Owasp.CsrfGuard.Carbon.xml` files in this repository itself, so that you don't have to worry about that.

##### 3. Data sourse configuration in `<DOCKERFILE_HOME>/base/files/wso2ei-6.1.1/conf/` (This should be done if you are changing the datasources)
- We have to change the `master-datasources.xml` inside the `datasources` folder according to wso2 documentation (https://docs.wso2.com/display/EI611/Configuring+master-datasources.xml).create schemas according to the names in `master-datasources.xml` and replace with your database server url in all places called `{your data source url}`.
 Replace `useName` and `Password` with your credentials.
 
- Same way `registry.xml` and `user-mgt.xml` should be changed according to wso2 documentations (https://docs.wso2.com/display/EI611/Configuring+registry.xml and https://docs.wso2.com/display/EI611/Configuring+user-mgt.xml).

### Configurations:

- Hostname. This is change because there should be a way to access to the WSO2 EI console in case of need. We don't need to access this through the LB.
 we are using this console only for mangment purpose. Users won't use this EI console. 
  
  ````
  <DOCKERFILE_HOME>/base/files/conf/carbon.xml
  <!--
    Host name or IP address of the machine hosting this server
    e.g. www.wso2.org, 192.168.1.10
    This is will become part of the End Point Reference of the
    services deployed on this server instance.
  -->
  <HostName>{ec2-instance-public-dns}</HostName> (NOT LB url)
  ````

- Management Hostname.  This is change because there should be a way to access to the WSO2 EI console in case of need. We don't need to access this through the LB.
 we are using this console only for mangment purpose. Users won't use this EI console. 
  ````
  <DOCKERFILE_HOME>/base/files/conf/carbon.xml
  <!--
    Host name to be used for the Carbon management console
    -->
  <MgtHostName>{ec2-instance-public-dns}</MgtHostName> (NOT LB url)
  ````

##### 4. Build the base docker image.
- For base, navigate to `<DOCKERFILE_HOME>/base` directory. <br>
  Execute `docker build` command as shown below.
    + `docker build -t wso2ei-base:6.1.1 .`
        
##### 5. Build docker images.
- For integrator, navigate to `<DOCKERFILE_HOME>/integrator` directory. <br>
  Execute `docker build` command as shown below. 
    + `docker build -t wso2ei-integrator:6.1.1 .`

### We need only upto this section for the ecs deployment process.###
    
##### 6. Running docker images.(run this if only need to test locally)
- For integrator,
    + `docker run -p 8280:8280 -p 8243:8243 -p 9443:9443 wso2ei-integrator:6.1.1`


##### 7. Accessing management console.
- For integrator,
    + `https:<DOCKER_HOST>:9443/carbon`


>In here, <DOCKER_HOST> refers to hostname or IP of the host machine on top of which containers are spawned.


## How to update configurations
Configurations would lie on the Docker host machine and they can be volume mounted to the container. <br>
As an example, steps required to change the port offset of integrator profile using `carbon.xml` is as follows.

##### 1. Stop the integrator container if it's already running.
In WSO2 Enterprise Integrator 6.1.1 product distribution, carbon.xml file for the integrator profile <br>
can be found at `<DISTRIBUTION_HOME>/conf`. Copy the file to some suitable location of the host machine, <br>
referred to as `<SOURCE_CONFIGS>/carbon.xml` and change the offset value under ports to 1.

##### 2. Grant read permission to `other` users for `<SOURCE_CONFIGS>/carbon.xml`
```
chmod o+r <SOURCE_CONFIGS>/carbon.xml
```

##### 3. Run the image by mounting the file to container as follows.
```
docker run 
-p 8281:8281 -p 8244:8244 -p 9444:9444
--volume <SOURCE_CONFIGS>/carbon.xml:<TARGET_CONFIGS>/carbon.xml
wso2ei-integrator:6.1.1
```

>In here, <TARGET_CONFIGS> refers to /home/wso2carbon/wso2ei-6.1.1/conf folder of the container.


## Docker command usage references

* [Docker build command reference](https://docs.docker.com/engine/reference/commandline/build/)
* [Docker run command reference](https://docs.docker.com/engine/reference/run/)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
