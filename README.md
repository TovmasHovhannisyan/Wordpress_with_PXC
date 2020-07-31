# Wordpress in Kubernetes

## Description

The project is created for learning and testing, and it is about deploying WordPress in Kubernetes. Also for the database, we will use the [Percona Xtradb Cluster](https://www.percona.com/doc/kubernetes-operator-for-pxc/kubernetes.html).
In the project, wordpress will be in multiple replicas in order to have high availability.

## Requirements
1. [Kubernetes cluster](https://linuxacademy.com/blog/containers/building-a-three-node-kubernetes-cluster-quick-guide/) (For minimum we need to have one master and 3 worker nodes )
2. One node for Haproxy and NFS
3. Storage Resources(Local)

## Steps for Deployment

> Please keep in mind that you need to put the necessary values inside YAML files in the project based on your environment structure.

### 1. Deploy Percona Xtradb Cluster 
  * First of all, we need to create the "pxc" namespace and make it as a default namespace, and create two [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/) inside Kubernetes with the following commands:
     ``` 
     $ kubectl create namespace pxc
     $ kubectl config set-context $(kubectl config current-context) --namespace=pxc
     
     $ kubectl apply -f storageClasses/proxysql-class.yaml 
     $ kubectl apply -f storageClasses/pxc-class.yaml
     ```

  * Next we need to create 9 [Persistant Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).
In our project, we use local persistent volumes, and to create them, we need to have 3 mounted partitions or disks on each node.*(You need to mention all mountpoins inside YAML files)*
     ```
     $ kubectl create -f persistent_voulumes/percona_pv_1.yaml
     ...
     $ kubectl create -f persistent_voulumes/percona_pv_9.yaml
     ```
    
  * Deploy  PXC with following commands:
  
     ```
     $ kubectl apply -f Deploy_for_PXC/crd.yaml
     $ kubectl apply -f Deploy_for_PXC/rbac.yaml
     $ kubectl apply -f Deploy_for_PXC/operator.yaml
     $ kubectl apply -f Deploy_for_PXC/secrets.yaml
     $ kubectl apply -f Deploy_for_PXC/cr.yaml
     ```
    > Please keep in mind that all necessary passwords are in the secrets.yaml file and encode in base64 format. You can check the values of that password decoding them  by using the command below:  
    
     ```
           $ echo -n "password" | base64
     ```    
### 2. Deploy Wordpress 
  As we mentioned earlier we use multiple replicas for wordpress pod to ensure high availability. To synchronize pods between each other we need to have one data source for all pods. In our case, we use shared NFS storage, where will be all our wordpress files.
  >*Follow the [link](https://www.tecmint.com/install-nfs-server-on-ubuntu/) to learn how to install and configure an NFS server*.  

Inside Kubernetes, we create services for WordPress in a way to give access to WordPress pods from the external network on port 80 on each node IP address on which WordPress pod will exist, and in our case, HAPROXY server is used to give access to that pods on one IP address (IP address of HAPROXY server) and to load balance traffic between pods.(You can find the Config file for Haproxy server in our project).  
To apply deployment you need to pass following steps:
  * Import all necessary values inside Wordpress_with_PXC/wordpress-deployment.yaml and persistent_voulumes/nfs.yaml files.
  * Execute command below:
   ```
   $ kubectl apply -f wordpress-deployment.yaml
   ```

As we use the official deployment project for the Percona Xtradb cluster for database service we have daily scheduled backups which will be stored on one of our "Available" PV. Besides that, we can create a backup manually. Please read the official [documentation](https://github.com/percona/percona-xtradb-cluster-operator/blob/v1.4.0/deploy/backup/README.md) to know how it works.