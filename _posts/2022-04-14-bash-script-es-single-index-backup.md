---
layout: post
title: Bash Script to take the single Elasticsearch index backup
author: Osni Theresa V X
date: 2022-03-09 11:00:00 +0800
categories: [Elasticsearch]
tags: [Elasticsearch, devops, ]
math: true
mermaid: true
description: Bash Script to take the single Elasticsearch index backup

---
#### Prerequisite
* You should have a ES domain configured

First, list the indices and decide the index you need to take the backup.

```
curl -XGET 'https://localhost:9200/_cat/indices' (Replace the URL with the exact endpoint)
```

Bash Script,

```
#!/bin/bash

#specify the index to backup
read -p "Enter the index name to take the backup:"  index
echo -en '\n'
echo "You have entered the index: $index"

#specify the snapshotsname
read -p "Enter the snapshot name:"  BACKUPNAME
echo -en '\n'
echo "You have entered the snapshot name: $BACKUPNAME"

#replace with your repo url and repository name
URL_REQ="https://localhost:9200/_snapshot/myrepo"

#include_global_state: to prevent the cluster global state to be stored as part of the snapshot

curl -XPUT "$URL_REQ/$BACKUPNAME?wait_for_completion=true&pretty" -H 'Content-Type: application/json' -d "{\"indices\":\"$index\",\"ignore_unavailable\":\"true\",\"include_global_state\":\"false\"}"
```
Run the bash script with the command

```
sh <script_name>.sh 
```
Note: Make sure your script file is having necessary permissions to run the script.

Verify it by listing the snapshot

```
curl -XGET 'https://localhost:9200/_snapshot/myrepo/_all?pretty'
```
Thats it. You single index backup is ready.
