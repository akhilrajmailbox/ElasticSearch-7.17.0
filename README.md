# ElasticSearch Cluster - Single node

## Create Docker image from Dockerfile

```bash
cd Docker
docker build -t akhilrajmailbox/elasticsearch:elasticsearch-7.17.0 . -f Dockerfile
```

## Docker Swarm Deployment

### Prerequisites

*  The client and daemon API must both be at least 1.24 to use this command. Use the docker version command on the client to check your client and daemon API versions.
* You need at least Docker Engine 19.03.0+ for this to work (Compose specification: 3.8).

```bash
docker version
```

### Run the following command to create a new swarm:

```bash
docker swarm init --default-addr-pool 172.18.0.0/16
```

### Create custom Docker network before deploying container (If the network is already created then ignore it)

*Note : Make sure that the subnet won't conflict with any other subnets*

```bash
docker network create --driver=overlay --subnet=172.30.0.0/16 my-swarm-network
```

### Deploy a stack to Swarm

```bash
docker stack deploy --compose-file=docker-swarm.yml es-stack
```

### Removing stack

```bash
docker stack rm es-stack
```

### Working with elastic cluster

* List the stack

```bash
docker stack ls
```

* List the tasks in the stack

```bash
docker stack ps es-stack
```

* List the services in the stack

```bash
docker stack services es-stack
```

* find the docker conatiner ID and Login to sample-app server

```bash
docker exec -it $(docker ps -q -f name=es-stack) /bin/bash
```

* Restarting sample-app

```bash
docker stack deploy --compose-file=docker-swarm.yml es-stack
```

* check the logs of sample-app server

```bash
docker logs -f $(docker ps -q -f name=es-stack)
```


## Docker Compose Deployment

### Prerequisites

* We are using `devops-network` docker network for all devops deployment.
* You need at least Docker Engine 17.06.0  and docker-compose 1.18 for this to work.

```bash
docker --version
docker-compose --version
```

### Create custom Docker network before deploying container (If the network is already created then ignore it)

*Note : Make sure that the subnet won't conflict with any other subnets*

```bash
docker network create --driver=bridge --subnet=172.31.0.0/16 devops-network
```

### Working with elastic cluster


* Login to elasticsearch

```bash
docker exec -it elasticsearch /bin/bash
```

* Login to kibana

```bash
docker exec -it kibana /bin/bash
```

* Restarting elasticsearch and kibana

```bash
docker-compose -f docker-compose.yml restart
```

* stopping/starting elasticsearch and kibana

```
docker-compose -f docker-compose.yml stop/start
```






ElasticSearch can access on this url : `http://<<es_client_loadbalancer_IPAddress>>:9200`

Kibana can access on this url : `http://<<es_client_loadbalancer_IPAddress>>:5601`


## The LOG Battle: Logstash and Fluentd

* source data processing pipeline
  * Logstash
  * Fluentd
* lightweight shippers
  * Filebeat
  * Fluentbit

[here](https://medium.com/tensult/the-log-battle-logstash-and-fluentd-c65f2f7c24b4)

### filebeat

[installation 1](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html)

[installation 2](https://crunchify.com/setup-guide-install-configure-filebeat/)

[configuration](https://www.elastic.co/guide/en/beats/filebeat/5.1/filebeat-configuration-details.html)


### Environment variables

This image can be configured by means of environment variables, that one can set on a `Deployment`.

* [CLUSTER_NAME](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#cluster.name)
* [NODE_NAME](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node.name)
* [NODE_MASTER](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#master-node)
* [NODE_DATA](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#data-node)
* [NETWORK_HOST](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#network-interface-values)
* [HTTP_CORS_ENABLE](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html#_settings_2)
* [HTTP_CORS_ALLOW_ORIGIN](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html#_settings_2)
* [MAX_LOCAL_STORAGE_NODES](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#max-local-storage-nodes)
* [ES_JAVA_OPTS](https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html)
* [ES_PLUGINS_INSTALL](https://www.elastic.co/guide/en/elasticsearch/plugins/current/installation.html) - comma separated list of Elasticsearch plugins to be installed. Example: `ES_PLUGINS_INSTALL="repository-gcs,x-pack"`
* [MEMORY_LOCK](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#bootstrap.memory_lock) - memory locking control - enable to prevent swap (default = `true`) .
* DISCOVERY_SERVICE (default = elasticsearch-discovery)
* [REPO_LOCATIONS](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html#_shared_file_system_repository) - list of registered repository locations. For example `"/backup"` (default = `[]`). The value of REPO_LOCATIONS is automatically wrapped within an `[]` and therefore should not be included in the variable declaration. To specify multiple repository locations simply specify a comma separated string for example `"/backup", "/backup2"`.
* [PROCESSORS](https://github.com/elastic/elasticsearch-definitive-guide/pull/679/files) - allow elasticsearch to optimize for the actual number of available cpus (must be an integer - default = 1)


### Backup

Mount a shared folder (for example via NFS) to `/backup` and make sure the `elasticsearch` user
has write access. Then, set the `REPO_LOCATIONS` environment variable to `"/backup"` and create
a backup repository:

`backup_repository.json`:
```
{
  "type": "fs",
  "settings": {
    "location": "/backup",
    "compress": true
  }
}
```

```bash
curl -XPOST http://<container_ip>:9200/_snapshot/nas_repository -d @backup_repository.json`
```

Now, you can take snapshots using:

```bash
curl -f -XPUT "http://<container_ip>:9200/_snapshot/nas_repository/snapshot_`date --utc +%Y_%m_%dt%H_%M`?wait_for_completion=true"
```



## azure blob storage and snapshots

For configuring the Azure blobe storage as snapshot registry, first create the blobe storage account in azure and then you have to configure the following environment variable for the es deployment


variable name	|	default value	| description
---------------------|---------------------|---------------------
AZURE_REPOSITORY_CONFIG	| -	| This value should be "true" to enable the snapshots |
AZURE_REPOSITORY_ACCOUNT_NAME	|	-	|	configure the blob storage name |
AZURE_REPOSITORY_ACCOUNT_KEY	|	-	|	configure the key1 or key2 value here |



## Google bucket and snapshots

variable name	|	default value	| description
---------------------|---------------------|---------------------
GCS_REPOSITORY_CONFIG	| -	| This value should be "true" to enable the snapshots |

**Note** : Create one service account json file and mount it to "/opt/secrets/serviceaccount.json"


## AWS s3 and snapshots

variable name	|	default value	| description
---------------------|---------------------|---------------------
S3_REPOSITORY_CONFIG	| -	| This value should be "true" to enable the snapshots |
S3_ACCESS_KEY	|	-	|	configure the access key for s3 |
S3_SECRET_KEY	|	-	|	configure the secret key for s3 |


### register the snapshot repository


#### Azure 

**note : i am using the sub container name `backup-container` in here, you can give any name here but you have to create the container in the storage account before starting the elasticsearch**

```
curl -XPUT 'http://192.168.0.12:9200/_snapshot/es_backup' -H 'Content-Type: application/json' -d '{
  "type": "azure",
  "settings": {
    "account": "default",
    "container": "backup-container",
    "base_path": "backups",
    "chunk_size": "32MB",
    "compress": true
  }
}'
```

## GCS

```
curl -XPUT 'http://192.168.0.12:9200/_snapshot/es_backup' -H 'Content-Type: application/json' -d '{
  "type": "gcs",
  "settings": {
    "bucket": "my_bucket",
    "client": "default",
    "base_path": "backups",
    "chunk_size": "32MB",
    "compress": true
  }
}'
```


## S3

```
curl -XPUT 'http://192.168.0.12:9200/_snapshot/es_backup' -H 'Content-Type: application/json' -d '{
  "type": "s3",
  "settings": {
    "bucket": "my_bucket",
    "client": "default",
    "base_path": "backups",
    "chunk_size": "32MB",
    "compress": true
  }
}'
```


### create a sample snapshot

```
curl -XPUT 'http://@192.168.0.12:9200/_snapshot/es_backup/snapshot_1' -H 'Content-Type: application/json' -d '{ "indices":"*","include_global_state":false }'
```


## Backup and Restore Steps

### Backup

* Install plugin:-

```
sudo ES_PATH_CONF=/etc/elasticsearch/es-node-2 /usr/share/elasticsearch/bin/elasticsearch-plugin install repository-azure
```

* change the configuration:-

```
sudo nano /etc/elasticsearch/es-node-2/elasticsearch.yml
cloud.azure.storage.default.account: xxxxxxxxxxx
cloud.azure.storage.default.key: xxxxxx
```

* Restart ES Service:-

```
sudo systemctl restart es-node-2_elasticsearch.service
```

* create Repo:-

```
curl -XPUT 'http://localhost:9200/_snapshot/azurebackup' -H 'Content-Type: application/json' -d '{ "type": "azure", "settings": { "container": "elasticsearch-snapshots", "base_path": "sunbirddevtele"} }'
```

* create Snapshot:-

```
curl -XPUT 'http://localhost:9200/_snapshot/azurebackup/snapshot_1' -H 'Content-Type: application/json' -d '{ "indices":"*","include_global_state":false }'
```

* check status of backup:-

```
curl -XGET 'http://localhost:9200/_snapshot/azurebackup/_all'
```


### Restore

* Install plugin:-

```
sudo ES_PATH_CONF=/etc/elasticsearch/es-node-2 /usr/share/elasticsearch/bin/elasticsearch-plugin install repository-azure
```

* change the configuration:-

```
sudo nano /etc/elasticsearch/es-node-2/elasticsearch.yml
cloud.azure.storage.default.account: xxxxxxxxxxx
cloud.azure.storage.default.key: xxxxxx
```

* Restart ES Service:- 

```
sudo systemctl restart es-node-1_elasticsearch.service
```

* create Repo:-

```
curl -XPUT 'http://localhost:9200/_snapshot/azurebackup' -H 'Content-Type: application/json' -d '{ "type": "azure", "settings": { "container": "elasticsearch-snapshots", "base_path": "sunbirddevtele"} }'
```

* Delete unwanted indices:- 

```
curl -XDELETE http://localhost:9200/_all
```

* Restore from snapshot:-

```
curl -XPOST 'http://localhost:9200/_snapshot/azurebackup/snapshot_1/_restore'
```


### Reference links ::

[es license and subscriptions](https://www.elastic.co/subscriptions)

[snapshots-restore](https://github.com/project-sunbird/sunbird-devops/wiki/elasticsearch-backup-and-restore-for-telemetry-and-composite-search-to-azure-blob)

[es-azure snapshots plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/6.8/repository-azure.html)

[es-azure plugin description](https://www.elastic.co/guide/en/elasticsearch/plugins/7.4/repository-azure-repository-settings.html)

[blob-storage and snapshots 1](https://azure.microsoft.com/en-in/blog/archive-elasticsearch-indices-to-azure-blob-storage-using-the-azure-cloud-plugin/)

[blob-storage and snapshots 2](https://stackoverflow.com/questions/54113059/elasticsearch-snapshot-creation-understanding-how-where-to-store-them-to)

