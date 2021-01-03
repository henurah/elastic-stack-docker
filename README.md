# elastic-stack-docker
Sample docker-compose files for different Docker configurations


sysctl -w vm.max_map_count=262144 

docker-compose -f create-certs.yml run --rm create_certs
docker-compose up -d

docker-compose down --verbose
docker volume rm es_certs


docker volume ls
docker-compose logs
