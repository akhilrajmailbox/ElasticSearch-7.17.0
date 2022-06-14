FROM docker.elastic.co/elasticsearch/elasticsearch:7.17.0
CMD ["bash"]
ENV ES_VERSION=7.17.0
ENV ES_BASE=/usr/share/elasticsearch
ENV PATH=${ES_BASE}/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

COPY elasticsearch.yml ${ES_BASE}/config/
RUN rm -rf ${ES_BASE}/modules/x-pack-ml

# Copy entrypoint script
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod a+x /usr/local/bin/docker-entrypoint.sh

# Set environment variables defaults
ENV ES_JAVA_OPTS="-Xms2g -Xmx2g"
ENV CLUSTER_NAME="my-cluster"
ENV NODE_NAME="my-node"
ENV MAX_LOCAL_STORAGE_NODES=1
ENV REPO_LOCATIONS=""
ENV NETWORK_HOST="0.0.0.0"
ENV HTTP_PORT="9200"
ENV TRANSPORT_PORT="9300"
ENV HTTP_CORS_ENABLE=true
ENV HTTP_CORS_ALLOW_ORIGIN=*
ENV MEMORY_LOCK=true
ENV NODE_MASTER=true
ENV NODE_DATA=true
ENV NODE_INGEST=true
ENV SECURITY_ENABLE=false
ENV MONITOR_ENABLE=false
ENV DISCOVERY_TYPE="single-node"
ENV AZURE_REPOSITORY_CONFIG=""
ENV AZURE_REPOSITORY_ACCOUNT_NAME=""
ENV AZURE_REPOSITORY_ACCOUNT_KE=""
ENV GCS_REPOSITORY_CONFIG=""
ENV GCP_SA_JSON_CRED=""
ENV S3_REPOSITORY_CONFIG=""
ENV S3_ACCESS_KEY=""
ENV S3_SECRET_KEY=""

ENTRYPOINT ["/bin/tini", "--", "/usr/local/bin/docker-entrypoint.sh"]
CMD ["eswrapper"]