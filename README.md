# EFK
Deploying elasticsearch - fluentd - kibana at k8s 

I use as a reference these tutorials and made some changes to fit my needs:

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes

https://blog.ptrk.io/how-to-deploy-an-efk-stack-to-kubernetes/

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
