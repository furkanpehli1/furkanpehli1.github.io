# Step by Step Configuring Elasticsearch Snapshot Feature

In this article I am going to explain how to activate snapshot feature in elasticsearch. Let's dive in.

In my case I use **Ceph S3 Storage** to store snapshots in an off-cluster storage location. But any cloud storage solution can also be used. For example:

- AWS S3
- Google Cloud Storage (GCS)
- Microsoft Azure

First of all bucket must be created and `secret` and `access` key must be retrieved. These keys will be added to clusters *keystore* in order to make our bucket accessible from cluster. But before adding our buckets key to cluster `repository-s3` plugin should be installed.

> Note that, on newest elasticsearch versions there is no need to install the plugin.

## 1. Installing the repository-s3 Plugin

To install plugin run following command on all nodes of elasticsearch cluster.

```
/usr/share/elasticsearch/bin/elasticsearch-plugin install repository-s3
```

After it is installed restart the `elasticsearch.service` to activate the plugin:

```bash
systemctl restart elasticsearch
```

When it is done, we need to add keys to our cluster which we created early for accessing bucket.

##  2. Registering the Cluster

In order to register cluster `access-key` and `secret-key` are needed. Luckily we got our keys while creating the s3 bucket. So we must add keys to `elastic-keystore`. To do that:

```bash
/usr/share/elasticsearch/bin/elasticsearch-keystore add s3.client.default.secret_key 
# Enter the keys when it is prompted
/usr/share/elasticsearch/bin/elasticsearch-keystore add s3.client.default.access_key
```

> Entering keys in any node will be enough.

After it handled successfully security settings of elastic must be reloaded. Following command will help us to do that:

```bash
curl -X POST "localhost:9200/_nodes/reload_secure_settings"
```

## 3. Creating Snapshot Repository and Initiating First Snapshot

In this step we will define our off-cluster bucket to clusters snapshot repository.

```bash
curl -X PUT "localhost:9200/_snapshot/REPOSITORY-NAME" -H 'Content-Type: application/json' -d'

{

"type": "s3",

"settings": {

"client": "default",

"bucket": "BUCKET-NAME",

"endpoint": "OBS-URL",

"path_style_access": "true",

"protocol": "https"

}

}

'
```

After it is done we are ready to create our first snapshot. Only one curl command is enough for that:

```bash
curl -X PUT "localhost:9200/_snapshot/REPOSITORY-NAME/my-first-snapshot-for-prod?wait_for_completion=true"
```

Conragulations you created your first snapshot 🙂 . Do you want to see it? If it is yes curl it 😁

```bash
curl -X GET "localhost:9200/_cat/snapshots/"
```

Isn't it fancy 🙃 . 

References:
- Elasticsearch's documentations.

Source:
- https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html
Published:
- https://usrbinpehli.medium.com/step-by-step-configuring-elasticsearch-snapshot-feature-7941af77042b | 19.04.22 - 21.45
