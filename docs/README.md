

# Setup

## Provision OpenShift 4.6+ cluster.
Provision 2 or more OpenShift 4.6 or later clusters on RHPDS, Cluster 1 will be used to host ACM and MongoDB the other clusters will be hosting the Pacman application.


## Fork the repository
You would need to create your own repository for ACM to consume the deployment files.
- 
 ```bash
  cd mongodb
  oc create namespace mongo
  oc apply -f mongo-deployment.yaml
  oc apply -f mongo-lb-service.yaml
  oc apply -f mongo-pvc.yaml
  oc apply -f mongo-route.yaml
  ```

## Deploy RHACM on the Cluster 1

Deploy RHACM on Cluster from operator hub
![Step 1](images/ACM_operator_01.png)

Follow the Steps and create a Default MultiClusterHub

![Step 2](images/ACM_operator_02.png)
## Deploy MongoDB at Cluster 1

Apply the policies under `mongodb/`.

  ```bash
  cd mongodb
  oc create namespace mongo
  oc apply -f mongo-deployment.yaml
  oc apply -f mongo-lb-service.yaml
  oc apply -f mongo-pvc.yaml
  oc apply -f mongo-route.yaml
  ```
## Edit the pacman deployment
* Get the service loadbalancer hostname
* Update the pacman deployment file pacman-deployment.yaml

  ```bash
  oc get service/mongo-lb -o=jsonpath='{.status.loadBalancer.ingress[0].hostname}'
  ```
   ```bash
  spec:
      hostNetwork: true
      containers:
      - image: quay.io/jpacker/nodejs-pacman-app:green
        env:
        - name: MONGO_SERVICE_HOST
          value: "UPDATE_SERVICE_LOADBALANCER_HOSTNAME_HERE"
        - name: MY_MONGO_PORT
          value: "27017"
        name: pacman
        ports:
        - containerPort: 8080
          name: http-server
  ```

## Cluster 2 setup
* Craete pacman-app namespace
* Allow higher privilege for default user in pacman-app namespace

```bash
  oc create namespace pacman-app
  oc project pacman-app
  oc adm policy add-scc-to-user privileged system:serviceaccount:pacman-app:default
```

## Deploy Application/Subscription/PlacementRule on Cluster 1
```bash
  oc apply -f pacman-app.yaml
  ```

## Add in the cluster label
Add the cluster label for Cluster 2 on ACM
### Steps: 
1. Click left menu > "Automated infrastructure" > "Clusters"
2. Click on the 3 Dots on the right
3. Input the Label
```bash
app.kubernetes.io/name=pacman
```
![Cluster_Label](images/ACM_Cluster_label_01.png)

## [WIP Ansible Integration]

### LoadBalancer Integration

Sample HAProxy configiration file (haproxy.cfg) to loadbalance between two or more clusters
* Edit file accordingly
```bash
backend app
    balance static-rr
    option httpchk GET / HTTP/1.1\r\nHost:\ {{ app domain name }}
    http-request set-header Host {{ app domain name }}
    
    mode http
      server aws {{ ocp console route }}:80 check inter 1s downinter 1s fall 1 rise 1 weight 10 #aws
      server aro {{ ocp console route }}:80 check inter 1s downinter 1s fall 1 rise 1 weight 10 #aro
```

## [WIP Tekton Integration]


# References

Github repos refrenced
- https://github.com/mdelder/k8s-pacman-app.git
- https://github.com/jnpacker/pacman
- [github.com/open-cluster-management/deploy](https://github.com/open-cluster-management/deploy)
- [Ansible Tower containerized install method](https://releases.ansible.com/ansible-tower/setup_openshift/)
- https://docs.ansible.com/ansible/latest/collections/awx/awx/tower_project_module.html
- https://github.com/ansible/awx
- https://github.com/ansible/awx/tree/devel/awx_collection#running
- https://developer.servicenow.com/
- [AWS Marketplace](https://aws.amazon.com/marketplace/pp/F5-Networks-F5-DNS-Load-Balancer-Cloud-Service/B07W3P8HM4)

