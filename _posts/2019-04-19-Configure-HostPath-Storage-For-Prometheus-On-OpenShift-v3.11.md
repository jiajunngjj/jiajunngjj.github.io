---
title: "Configure HostPath Storage For Prometheus on OpenShift v3.11"
tags: 
  - redhat
  - openshift
  - prometheus
  - openshift monitoring
permalink: /:year/:month/:day/prometheus-openshift
---
This post covers the configuration of persistent storage with HostPath for OpenShift Monitoring stack.
 
In OpenShift v3.11, Prometheus cluster monitoring is now fully supported and deployed by default. As the monitoring stack is fairly new, there are many on-going discussions on supported storage plugins. And the documentation does not cover the detailed steps of setting up the storage. The configurability is also limited with Operator in placed which "protects" the configuration. 
 
Through various implementations with HostPath requirement, I have figured a way to make HostPath storage works with Operator.  

## Configuration Steps
### Assumptions
- HostPath SCC is already created
- HostPath SCC set to allow hostpath directory volume plugin
- Granting access to this SCC to all users

### Inventory file
It is mandatory set the following variables in the inventory file:

| Variable  | Description |
| ------------- | ------------- |
| openshift_cluster_monitoring_operator_node_selector={'role':'metrics'}  | I have created a label, "role=metrics" to land the pods on the dedicated nodes for Prometheus/Alertmanager. These nodes should contain the HostPath for persistent storage.  |
| openshift_cluster_monitoring_operator_prometheus_storage_capacity=50G  | The persistent volume claim size for each of the Prometheus instances. This variable applies only if openshift_cluster_monitoring_operator_prometheus_storage_enabled is set to true. Defaults to 50Gi.|
| openshift_cluster_monitoring_operator_alertmanager_storage_capacity=5Gi | The persistent volume claim size for each of the Alertmanager instances. This variable applies only if openshift_cluster_monitoring_operator_alertmanager_storage_enabled is set to true. Defaults to 2Gi. |
| openshift_cluster_monitoring_operator_prometheus_storage_enabled=true | Enable persistent storage for Prometheus |
| openshift_cluster_monitoring_operator_alertmanager_storage_enabled=true | Enable persistent storage for Alertmanager |

### Run the playbook
```
$ ansible-playbook -i <INVENTORY>  /usr/share/ansible/openshift-ansible/playbooks/openshift-monitoring/config.yml```
```

### Node hostpath configuration
- Find the namespace ID:
  ```
  $ oc get namespace openshift-monitoring -o yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    annotations:
      openshift.io/description: Openshift Monitoring
      openshift.io/display-name: ""
      openshift.io/node-selector: ""
      openshift.io/sa.scc.mcs: s0:c11,c0
      openshift.io/sa.scc.supplemental-groups: 1000110000/10000
      openshift.io/sa.scc.uid-range: 1000110000/10000
    creationTimestamp: 2019-04-29T03:48:23Z
    labels:
      openshift.io/cluster-monitoring: "true"
    name: openshift-monitoring
    resourceVersion: "15977564"
    selfLink: /api/v1/namespaces/openshift-monitoring
    uid: a3173b3e-6a31-11e9-b915-000c29451a06
  spec:
    finalizers:
    - kubernetes
  status:
    phase: Active 
  ```
  
In the example above, my namespace ID is 1000110000.

- Change ownership and SELinux label of the HostPath:
    ```
    $ chown 1000110000:root /metrics/
    $ semanage fcontext -a -t svirt_sandbox_file_t /metrics
    $ restorecon -v /metrics/
    $ chown 1000110000:root /alertmanager/
    $ semanage fcontext -a -t svirt_sandbox_file_t /alertmanager
    $ restorecon -v /alertmanager/
    ```
  
### Post Installation - Prometheus
After the playbook has completed successfully, you will see that the pods are starting:
    
 ```
 $ oc get pods
 NAME                                          READY     STATUS    RESTARTS   AGE
 cluster-monitoring-operator-6566bf44b-sb8wk   1/1       Running   0          1m
 prometheus-operator-77bc9b6c68-gw92h          1/1       Running   0          28s
  
 NAME                                          READY     STATUS    RESTARTS   AGE
 cluster-monitoring-operator-6566bf44b-sb8wk   1/1       Running   0          2m
 grafana-67cb69b946-684vc                      2/2       Running   0          55s
 prometheus-k8s-0                              0/4       Pending   0          5s
 prometheus-operator-77bc9b6c68-gw92h          1/1       Running   0          1m
 ```

By default, when openshift_cluster_monitoring_operator_prometheus_storage_enabled is set to true, it will create a default storage class and Prometheus-k8s-0 will remain in Pending state until it has successfully claims its PV. 

- The first thing to do is to delete the PVC created by deployment:
    ```
    $ oc get pvc
    NAME                                 STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    prometheus-k8s-db-prometheus-k8s-0   Pending                                                      4m

    $ oc delete pvc prometheus-k8s-db-prometheus-k8s-0
    persistentvolumeclaim "prometheus-k8s-db-prometheus-k8s-0" deleted
    ```
  
- Create the new PV for Prometheus: 
    ``` 
    $ vi pv-prometheus-0.yml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: prometheus-k8s-db-prometheus-k8s-0
    spec:
      capacity:
        storage: 100G
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: /metrics
      persistentVolumeReclaimPolicy: Retain
    ClaimRef:
      name: prometheus-k8s-db-prometheus-k8s-0
      namespace: openshift-monitoring

    $ oc create -f pv-prometheus-0.yml
    ```
Note that /metrics is the HostPath attached to the node.

- Create the new PVC for Prometheus:
    ```
    $ vi pvc-prometheus-0.yml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: prometheus-k8s-db-prometheus-k8s-0
    spec:
      resources:
        requests:
          storage: 100G
      accessModes:
        - ReadWriteOnce

    $ oc create -f pvc-prometheus-0.yml -n openshift-monitoring
    ```
    
- After this is done, the second Prometheus pod will start to spin up. So, repeat the steps 1 to 3, creating a new PV and PVC with prometheus-k8s-db-prometheus-k8s-1:
    ```
    $ vi pv-prometheus-1.yml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: prometheus-k8s-db-prometheus-k8s-1
    spec:
      capacity:
        storage: 100G
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: /metrics
      persistentVolumeReclaimPolicy: Retain
    ClaimRef:
      name: prometheus-k8s-db-prometheus-k8s-1
      namespace: openshift-monitoring


    $ vi pvc-prometheus-1.yml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: prometheus-k8s-db-prometheus-k8s-1
    spec:
      resources:
        requests:
          storage: 100G
    accessModes:
      - ReadWriteOnce
    ```
  
### Post Installation - Alertmanager
After two Prometheus pods are spun up successfully, Alertmanager pods will be spinning up right after.
  ```
  $ oc get pods
  NAME                                          READY     STATUS    RESTARTS   AGE
  alertmanager-main-0                           0/3       Pending   0          42s
  cluster-monitoring-operator-6566bf44b-sb8wk   1/1       Running   0          14m
  grafana-67cb69b946-684vc                      2/2       Running   0          13m
  prometheus-k8s-0                              4/4       Running   1          1m
  prometheus-k8s-1                              4/4       Running   3          6m
  prometheus-operator-77bc9b6c68-gw92h          1/1       Running   0          13m
  ```
  
Again, the alertmanager will remain at Pending status until it has claimed the PV successfully.


- Delete the Alertmanager PVC created by deployment:
    ```
    $ oc get pvc
        NAME                                       STATUS    VOLUME                               CAPACITY   ACCESS MODES   STORAGECLASS   AGE
        alertmanager-main-db-alertmanager-main-0   Pending                                                                                 41s
        prometheus-k8s-db-prometheus-k8s-0         Bound     prometheus-k8s-db-prometheus-k8s-0   100G       RWO                           7m
        prometheus-k8s-db-prometheus-k8s-1         Bound     prometheus-k8s-db-prometheus-k8s-1   100G       RWO                           4m

   $ oc delete pvc alertmanager-main-db-alertmanager-main-0
   persistentvolumeclaim "alertmanager-main-db-alertmanager-main-0" deleted
   ```
- Create the new PV for Alertmanager:
    ```
    $ vi pv-alertmanager-0.yml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: alertmanager-main-db-alertmanager-main-0
    spec:
      capacity:
        storage: 10G
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: /alertmanager
      persistentVolumeReclaimPolicy: Retain
    storageClassName: alertmanager-storageclass
    ClaimRef:
      name: alertmanager-main-db-alertmanager-main-0
      namespace: openshift-monitoring

    $ oc create -f pv-alertmanager-0.yml
    ```
Note that /alertmanager is the HostPath attached to the node.

- Create the PVC for Alertmanager:
    ```
    $ vi pvc-alertmanager-0.yml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: alertmanager-main-db-alertmanager-main-0
    spec:
      resources:
        requests:
          storage: 10G
      accessModes:
        - ReadWriteOnce

    $ oc create -f pvc-alertmanager-0.yml -n openshift-monitoring
    ```
    
After this is done, the second and third Alertmanager pod will start to spin up. So, repeat the steps 1 to 3, creating a new PV and PVC with alertmanager-main-db-alertmanager-main-1 and alertmanager-main-db-alertmanager-main-2 respectively.

The end result for the PV and PVC in OpenShift-Monitoring:

  ``` 
  $ oc get pv
  NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                                                       STORAGECLASS   REASON    AGE
  alertmanager-main-db-alertmanager-main-0   10G        RWO            Retain           Bound     openshift-monitoring/alertmanager-main-db-alertmanager-main-0                            5m
  alertmanager-main-db-alertmanager-main-1   10G        RWO            Retain           Bound     openshift-monitoring/alertmanager-main-db-alertmanager-main-1                            1m
  alertmanager-main-db-alertmanager-main-2   10G        RWO            Retain           Bound     openshift-monitoring/alertmanager-main-db-alertmanager-main-2                            8s
  prometheus-k8s-db-prometheus-k8s-0         100G       RWO            Retain           Bound     openshift-monitoring/prometheus-k8s-db-prometheus-k8s-0                                  13m
  prometheus-k8s-db-prometheus-k8s-1         100G       RWO            Retain           Bound     openshift-monitoring/prometheus-k8s-db-prometheus-k8s-1                                  11m

  $ oc get pvc
  NAME                                       STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  alertmanager-main-db-alertmanager-main-0   Bound     alertmanager-main-db-alertmanager-main-0   10G        RWO                           3m
  alertmanager-main-db-alertmanager-main-1   Bound     alertmanager-main-db-alertmanager-main-1   10G        RWO                           1m
  alertmanager-main-db-alertmanager-main-2   Bound     alertmanager-main-db-alertmanager-main-2   10G        RWO                           6s
  prometheus-k8s-db-prometheus-k8s-0         Bound     prometheus-k8s-db-prometheus-k8s-0         100G       RWO                           13m
  prometheus-k8s-db-prometheus-k8s-1         Bound     prometheus-k8s-db-prometheus-k8s-1         100G       RWO                           10m
  ```
    
### References
* [https://docs.openshift.com/container-platform/3.11/install_config/prometheus_cluster_monitoring.html#persistent-storage](https://docs.openshift.com/container-platform/3.11/install_config/prometheus_cluster_monitoring.html#persistent-storage)
