# Fluentd

## How to see the full log of a container in a pod?

### Figure out the node where the pod is running
```zsh
kgp jerry-jaeger-es-index-cleaner-4-nbc52 -o wide

NAME                                    READY   STATUS   RESTARTS   AGE   IP            NODE                                         NOMINATED NODE   READINESS GATES
jerry-jaeger-es-index-cleaner-4-nbc52   0/1     Error    0          30m   10.85.93.40   ip-10-85-24-102.us-west-2.compute.internal   <none>           <none>
```
It's `ip-10-85-24-102.us-west-2.compute.internal` in this case.

### Figure out the Fluentd pod running on the the same node

Do a search using the node name i.e. `ip-10-85-24-102.us-west-2.compute.internal` in the "Infrastructure List" in Datadog.
Click on the node name and switch to the "Kubernetes" tab in the right side panel.
Figure out the Fluentd pod, it's `fluentd-main-production-red-usw2-5m52h` in this case.

### See the full log of the container

Execute `bash` in the `fluentd` container
```zsh
k exec fluentd-main-production-red-usw2-5m52h -c fluentd -i -t -- bash
```
Locate the `.log` files for the containers included in the `jerry-jaeger-es-index-cleaner-4-nbc52` pod under `/var/log/containers/`.
