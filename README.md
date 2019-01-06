# EFK
Deploying elasticsearch - fluentd - kibana at k8s 

I use as a reference these tutorials and made some changes to fit my needs:

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes

https://blog.ptrk.io/how-to-deploy-an-efk-stack-to-kubernetes/

## Create namespace and storage class objects (you'll need them):

Create kube-logging namespace:
```
kubectl create -f ns-kube-logging.yaml
```

Create storageclass object:
```
kubectl create -f sc.yaml
```

Note: I'm on AWS, change "provisioner" value accordingly. 
