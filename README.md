# EFK
Deploying elasticsearch - fluentd - kibana at k8s 

I use as a reference these tutorials and made some changes to fit my needs:

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes

https://blog.ptrk.io/how-to-deploy-an-efk-stack-to-kubernetes/

This is how Kibana looks, showing stdout of "counter" app:
![Image description](https://github.com/pulpbill/EFK/blob/master/kibana.JPG)

### Create namespace and storage class objects (you'll need them):

Create kube-logging namespace:
```
kubectl create -f ns-kube-logging.yaml
```

Create storageclass object:
```
kubectl create -f sc.yaml
```

Note: I'm on AWS, change "provisioner" value accordingly. 

### For Kibana auth, you'll need to create a secret:

- First, generate the hashed passwd (I use admin as user):
```
htpasswd -c ./auth admin
```
- Create secret: 
```
kubectl -n kube-logging create secret generic kibana-basic-auth --from-file auth
```

*If you don't have htpasswd installed, try looking for the package, I'm on red hat based AMI:
```
yum whatprovides */htpasswd
```

### Optional: Deploy minimal counter Pod that prints sequential numbers to stdout:
```
kubectl create -f counter.yaml
```
Notice that this pod will run at the default namespace since we didn't specify one and Kibana still can see its logs. In Kubernetes, containerized applications that log to stdout and stderr have their log streams captured and redirected to JSON files on the nodes. The Fluentd Pod will tail these log files, filter log events, transform the log data, and ship it off to the Elasticsearch logging backend we deployed. In addition to container logs, the Fluentd agent will tail Kubernetes system component logs like kubelet, kube-proxy, and Docker logs.     

## Using AWS EFS:

In order to use a NFS, AWS EFS in this case, we need to follow this steps:

- Create an EFS: 
```
aws efs create-file-system --creation-token efs-for-testing
```

- Get details about your EC2 instances (K8s nodes):
```
aws ec2 describe-instances --filters &lt;your-filters-to-retrieve-k8s-nodes&gt;
```

- Create a mount target (you need to do it for all your subnets/SG):
```
aws efs create-mount-target \
--file-system-id <your-efs-id> \
--subnet-id {SubnetId} \
--security-groups {SecurityGroupId}
```
We need to add a deployment which will set up an EFS-Provisioner (https://github.com/kubernetes-incubator/external-storage/tree/master/aws/efs) in our cluster:

- Create the necessary Kubernetes objects:
```
kubectl create -f EFS/ -n kube-logging
```
Note: This will create a StorageClass - PersistenVolume - PersistenVolumeClain and RBAC objects so you can make use of your EFS.
Make sure to update the namespace, efs id and aws region. (I use EFK at kube-logging)


Source (I made some changes): https://www.juandebravo.com/2018/09/28/aws-efs-kubernetes/ 
