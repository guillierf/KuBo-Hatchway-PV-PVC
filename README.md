# KuBo-Hatchway-PV-PVC

Hatchway is an open source project that provides support of PV/PVC for K8s

##

## Configure KuBo

On KuBo client VM:
Create the following file: /root/kubo-deployment/manifests/ops-files/vsphere-cloud-provider.yml.

/root/kubo-deployment/manifests/ops-files/vsphere-cloud-provider.yml:
```
- type: replace
  path: /instance_groups/name=master/jobs/name=cloud-provider/properties/cloud-provider?
  value: &vsphere_cloud_provider
   type: vsphere
   vsphere:
     user: 'administrator@vsphere.local'
     password: 'VMware1!'
     server: '10.40.206.61'
     insecure-flag: 1
     datacenter: 'Datacenter'
     datastore: 'NFS-LAB-DATASTORE'
     working-dir: '/Datacenter/vm/kubobosh-vms'
- type: replace
  path: /instance_groups/name=worker/jobs/name=cloud-provider/properties/cloud-provider?
  value: *vsphere_cloud_provider
  ```
  
  Modify the 13-create-kubo-deployment-1.sh script to include the vsphere-cloud-provider.yml file:
  
  13-create-kubo-deployment-1.sh:
  ```
  /usr/local/bin/bosh int manifests/kubo.yml \
     -o manifests/ops-files/master-haproxy-vsphere.yml \
     -o manifests/ops-files/worker-haproxy.yml \
     -o manifests/ops-files/vsphere-cloud-provider.yml \
     -v deployments_network=KUBO-PG-1 \
     -v kubo-admin-password="VMware1!" \
     -v kubelet-password="VMware1!" \
     -v kubernetes_master_port=443 \
     -v kubernetes_master_host=10.40.207.46 \
     -v deployment_name=mykubocluster-1 \
     -v worker_haproxy_tcp_frontend_port=1234 \
     -v worker_haproxy_tcp_backend_port=4231 > mykubo-1.yml
  ```
     
  
  Deploy the K8s using the typical process.
  
  ## Create kubevols directory on the datastore
  
  on datastore NFS-LAB-DATASTORE, we need to create the directory kubevols.
  
  SSH to ESXi host
  ```
   cd /vmfs
   cd volumes/
   cd NFS-LAB-DATASTORE/
   mkdir kubevols
 ```
 
 ## Deploy the GuestBook application
 
 
Deploy Redis Storage Class:
```
kubectl create -f redis-sc.yaml
```

Check:
```
kubectl get sc
```

Deploy Redis Master and Redis Slave PV:
```
kubectl create -f redis-master-pv.yaml -f redis-slave-pv.yaml
```

Check:
```
kubectl get pv
```

Deploy Redis Master and Redis Slave PVC:
```
kubectl create -f redis-master-claim.yaml -f redis-slave-claim.yaml
```

Check:
```
kubectl get pvc
```

Deploy GuestBook application:
```
kubectl create -f guestbook-all-in-one.yaml
```

Check:
```
kubectl get pod
kubectl get svc
```

To connect to GuestBook application (the application uses K8s Service with type=NodePort):

http://[node IP]:[node Port]

