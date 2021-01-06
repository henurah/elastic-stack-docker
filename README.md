# elastic-stack-docker
Sample docker-compose files for different Docker configurations with SSL-enabled (self-signed certificates).  Designed to run on a single host.

## Overview




## Prerequisites
[Docker](https://docs.docker.com/get-docker/) and [docker-compose](https://docs.docker.com/compose/install/) will need to be installed.  Also Elasticsearch requires the system to pass [bootstrap checks](https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html) when not set up in single-node discovery mode.  For my install on a RHEL7 VM in GCP, I had to increase the maximum map count check on my system: 

`sysctl -w vm.max_map_count=262144` 


## Running the 
docker-compose -f create-certs.yml run --rm create_certs

docker-compose up -d


docker-compose down -v

docker volume rm es_certs


docker volume ls

docker-compose logs



https://raw.githubusercontent.com/elastic/apm-server/7.x/apm-server.docker.yml


https://www.elastic.co/guide/en/apm/server/current/api-key.html#create-api-key-workflow
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



