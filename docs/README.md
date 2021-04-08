

# Setup

## Provision OpenShift 4.6+ cluster.
Provision 2 or more OpenShift 4.6 or later clusters on RHPDS, 1 Cluster will be used to host ACM and MongoDB the other clusters will be hosting the Pacman application.


## Deploy RHACM on the Cluster 1

Deploy RHACM on Cluster from operator hub

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
Get the service loadbalancer hostname and update the pacman deployment file pacman-deployment.yaml
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

```bash
  oc create namespace pacman-app
  oc project pacman-app
  oc adm policy add-scc-to-user privileged system:serviceaccount:pacman-app:default
```

## Deploy Application/Subscription/PlacementRule on Cluster 1
```bash
  oc apply -f pacman-app.yaml
  ```

## Add in the cluster label for Cluster 2
```bash
app.kubernetes.io/name=pacman
```

# References

- [github.com/open-cluster-management/deploy](https://github.com/open-cluster-management/deploy)
- [Ansible Tower containerized install method](https://releases.ansible.com/ansible-tower/setup_openshift/)
- https://docs.ansible.com/ansible/latest/collections/awx/awx/tower_project_module.html
- https://github.com/ansible/awx
- https://github.com/ansible/awx/tree/devel/awx_collection#running
- https://developer.servicenow.com/
- [AWS Marketplace](https://aws.amazon.com/marketplace/pp/F5-Networks-F5-DNS-Load-Balancer-Cloud-Service/B07W3P8HM4)
