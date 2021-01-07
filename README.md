# elastic-stack-docker
Sample docker-compose files for different Docker configurations with SSL-enabled (self-signed certificates).  Designed to run on a single host to illustrate HA/FT capabilities with SSL/TLS for POC purposes (not for production use unless revised to have instances run on separate underlying hosts).

## Overview
The following configurations are provided for you to quickly set up an Elastic Stack highlighting the HA/FT and SSL/TLS configurations.

* [Kibana](Kibana) - 3 node ES cluster with 1 Kibana instance
* [HA-FT](HA-FT) - 3 node ES cluster with 2 Kibana instances
* [APM_HA-FT](APM_HA_FT) - 3 node ES cluster with 2 Kibana instances and 2 APM Servers (the main focus of this README)

Please see the [official documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-tls-docker.html) for further information that may not be addressed in this README.

## Prerequisites
[Docker](https://docs.docker.com/get-docker/) and [docker-compose](https://docs.docker.com/compose/install/) will need to be installed.  Also Elasticsearch requires the system to pass [bootstrap checks](https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html) when not set up in single-node discovery (development) mode.  For an install on a RHEL7 VM in GCP, the maximum map count check on the system had to be increased using the following command: 

`sysctl -w vm.max_map_count=262144` 


## Running the Scripts
* Change to the directory of the deployment of your choosing and update the [.env](APM_HA-FT/.env) file.  The defaults can be kept for testing purposes.
* Create the SSL certs for your instances (may need to modify the [instances.yml](APM_HA-FT/instances.yml) file):
`docker-compose -f create-certs.yml run --rm create_certs`
* Start your containers using the `docker-compose` command:
`docker-compose up -d`
* Please note the ports that are being exposed on the host from the [docker-compose.yml](APM_HA-FT/docker-compose.yml) file.  For example, although internally the containers for Elasticsearch are listening on port 9200, they are mapped to 9201, 9202, and 9203 on the host, respectively. 
* Verify that the Kibana host(s) are accessible (set to listen on port 5601 and 5602 on the host) and that you can log into the Kibana hosts using the _elastic_ superuser and password (set in the .env file).
* Verify that the APM Server is properly configured by going into the APM application in Kibana and going through the wizard up to the point of the APM Server set up.
* Install and configure your APM agents to send data to the APM Server(s).  Java Agent example is shown in the next section.
* Delete containers when done:
`docker-compose down -v`
* Remove Docker storage volumes (in case the -v flag wasn't used in the previous step):
`docker volume ls`,
`docker volume rm es_certs`,
...
* Show logs for troubleshooting:
`docker-compose logs`

## Setting Up and Testing Java Agent Using Spring Boot Example
Please refer to the [official documentation](https://www.elastic.co/guide/en/apm/agent/java/current/setup-javaagent.html) for more information.  We will set up the Java Agent for the official [Spring Boot](https://spring.io/guides/gs/spring-boot/) example using Gradle.

* Download the latest Elastic Java APM agent ([official documentation](https://www.elastic.co/guide/en/apm/agent/java/current/setup-javaagent.html))
* The APM Server in [APM_HA-FT/] has been configured to use API keys for [agent authentication](https://www.elastic.co/guide/en/apm/server/current/api-key.html#create-api-key-workflow).  Please run the following in the _Dev Tools_ application in Kibana:
```
POST /_security/api_key
{
  "name": "java-001", 
  "expiration": "1d", 
  "role_descriptors": {
    "apm": {
      "applications": [
        {
          "application": "apm",
          "privileges": ["sourcemap:write", "event:write", "config_agent:read"], 
          "resources": ["*"]
        }
      ]
    }
  }
}
```
* Copy the output from the _POST_ command.  It should look similar to the following:
```
{
  "id" : "maoB23YBYPQvC_KeVSUR",
  "name" : "java-001",
  "expiration" : 1610078466284,
  "api_key" : "KU8o6aY-RKWcF_hBRgAayw"
} 
```
* Base64 encode the string that consists of the _id_ and _api_key_ with a colon delimiter:
```
$ echo -n maoB23YBYPQvC_KeVSUR:KU8o6aY-RKWcF_hBRgAayw |base64
bWFvQjIzWUJZUFF2Q19LZVZTVVI6S1U4bzZhWS1SS1djRl9oQlJnQWF5dw==
```
* Add the following lines to the Spring Boot build.gradle file substituting the base64-encoded api_key and the urls for the APM Server:
```
bootRun {
        jvmArgs = ["-javaagent:./elastic-apm-agent-1.19.0.jar", "-Delastic.apm.api_key=bWFvQjIzWUJZUFF2Q19LZVZTVVI6S1U4bzZhWS1SS1djRl9oQlJnQWF5dw==", "-Delastic.apm.verify_server_cert=false", "-Delastic.apm.service_name=your-example-service", "-Delastic.apm.application_packages=org.example", "-Delastic.apm.server_urls=https://pathToHost:8201,https://pathToHost:8202"]
}
```
* Run the example Spring Boot program: 
`./gradlew bootRun`
* Test the HTTP endpoint of the example program:
`curl localhost:8080`
* Check the APM application in Kibana.  You should your service listed with a single transaction.
* Please see the [APM YAML file](https://raw.githubusercontent.com/elastic/apm-server/7.x/apm-server.docker.yml) or the [official documentation](https://www.elastic.co/guide/en/apm/agent/java/current/configuration.html) for more information on configuring the Java APM agent.






