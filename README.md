# hdfs

Follow along below to get the setup working on your cluster.

## Prerequisites

* Ensure that you have the [hasura cli](https://docs.hasura.io/0.15/manual/install-hasura-cli.html) tool installed on your system.

```sh
$ hasura version
```

Once you have installed the hasura cli tool, login to your Hasura account

```sh
$ # Login if you haven't already
$ hasura login
```

* Ensure that you have [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)  installed on your system.

```sh
$ kubectl version
```

* You should also have [git](https://git-scm.com) installed.

```sh
$ git --version
```

## Getting started

```sh
$ # Run the quickstart command to get the project
$ hasura quickstart hasura/hdfs
```

Note the name of the cluster printed in the output.

```sh
$ # Navigate into the Project
$ cd hdfs
```
## Deploy app

Deployment is a 3-step process.  
   

Step-1: Deploy Persistent Volumes and Persistent Volume Claims (https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

```sh
$ # Ensure that you are in the hdfs directory
$ cd custom_k8s/pv
$ kubectl create -f k8s.yaml --context=<cluster_name>
```

Step-2: Deploy Namenode 

```sh
$ # Ensure that you are in the hdfs directory
$ cd custom_k8s/namenode
$ kubectl create -f k8s.yaml --context=<cluster_name>
```

Step-3: Deploy Datanodes

```sh
$ # Ensure that you are in the hdfs directory
$ cd custom_k8s/datanode
$ kubectl create -f k8s.yaml --context=<cluster_name>
```

Optional Step: Deploy Namenode web UI

To get a HTTPS endpoint to view your Namenode web UI. Append the following snippet to your `routes.yaml`

```sh
namenode:
  /:
    corsPolicy: allow_all
    upstreamService:
      name: namenode
      namespace: '{{ cluster.metadata.namespaces.user }}'
    upstreamServicePath: /
    upstreamServicePort: 80
```

The upstream service `namenode` was already deployed as part of Step-2 above.

```sh
$ # Ensure that you are in the hdfs directory
$ cd conf
$ # Add above snippet to the end of routes.yaml
$ vim routes.yaml
$ # Commit changes and push
$ git add . && git commit -m "add namenode ui route"
$ git push hasura master
```

Goto `namenode.<cluster_name>.hasura-app.io` to access the Namenode web UI.

## Explore HDFS

Exec into `namenode-0` pod to run commands using `hdfs` client. 

```sh
$ kubectl exec -it namenode-0  --context=<cluster_name> -- /bin/bash

root> hdfs dfs -put  /etc/hosts /
root> hdfs dfs -ls /
```

And thats it!

