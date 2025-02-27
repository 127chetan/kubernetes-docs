---
reviewers:
- eparis
- pmorie
title: Configure Redis with a ConfigMap
content_type: tutorial
weight: 30
---

<!-- overview -->

In this tutorial, we will walk through a real world example of how to configure Redis using a ConfigMap. See [Configure a Pod to Use a ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) for more information about ConfigMaps. 
---
## What you'll learn

In this tutorial, you’ll learn how to do the following tasks:

* Create a ConfigMap with Redis configuration values.
* Create a Redis Pod that mounts and uses the created ConfigMap.
* Verify that the configuration is correctly applied.
---
## {{% heading "Requirements" %}}
{{< include "task-tutorial-prereqs.md" >}}

### kubectl-specific requirements
* You've installed [kubectl 1.14](https://kubernetes.io/releases/download/) or higher.
* You've completed the [Configure a Pod to Use a ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) tutorial.
---

<!-- lessoncontent -->

## Step 1: Create a ConfigMap
1. Run the following commands to create a ConfigMap with an empty configuration block:
```shell
cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: ""
EOF
```

2. Run the following commands to apply the created ConfigMap with a Redis Pod manifest:
```shell
kubectl apply -f example-redis-config.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

## Step 2: Review your created artifacts
1. Review the Redis Pod manifest for the following information:
* Under `volumes`, a new `config` volume is created.
* On the `config` volume, the `key` and `path` values tell us that the `redis-config` key is shared with the `example-redis-config` ConfigMap as a file named `redis.conf`.
* The `config` volume is mounted at `/redis-master`.

#### Example redis-pod.yaml file
{{% code_sample file="pods/config/redis-pod.yaml" %}}

2. Run the following command to review the created objects:
```shell
kubectl get pod/redis configmap/example-redis-config 
```
#### Response
```shell
NAME        READY   STATUS    RESTARTS   AGE
pod/redis   1/1     Running   0          8s

NAME                             DATA   AGE
configmap/example-redis-config   1      14s
```

3. Run the following commaand to confirm that the `redis-config` key is blank in the `example-redis-config` ConfigMap:
```shell
kubectl describe configmap/example-redis-config
```
#### Response
```shell
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
```

4. Run the following command to enter the Redis Pod and to check the current configuration:
```shell
kubectl exec -it redis -- redis-cli
```

5. Run the following command to to confirm that your Redis `maxmemory` value is set to the default value of `0`:
```shell
127.0.0.1:6379> CONFIG GET maxmemory
```

6. Run the following command to confirm that your Redis `maxmemory-policy` value is set to the default value of `noeviction`:
```shell
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

{{< alert title="Note" color="info" >}}

`maxmemory` is a Redis configuration that enables you to set the memory limit at which your eviction policy takes effect. Similarly, `maxmemory-policy` enables you to set the eviction policy when the limit set by `maxmemory` is reached. See [https://redis.io/docs/latest/develop/reference/eviction/] in the Redis documentation to learn more key eviction.

{{< /alert >}}

## Step 3: Add configuration values to the ConfigMap
1. Replace the `example-redis-config` ConfigMap with the following code:
{{% code_sample file="pods/config/example-redis-config.yaml" %}}

2. Run the following command to apply the updated ConfigMap:
```shell
kubectl apply -f example-redis-config.yaml
```

3. Run the following command to confirm that the ConfigMap is updated:
```shell
kubectl describe configmap/example-redis-config
```

#### Response
```shell
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----
maxmemory 2mb
maxmemory-policy allkeys-lru
```

{{< alert title="Note" color="info" >}}

While the ConfigMap is updated, the configuration values are not applied to the Redis Pod. You must restart (delete and recreate) the Pod to get the updated ConfigMap values.

{{< /alert >}}

## Step 4: Delete and recreate the Redis Pod
1. Run the following commands to delete and recreate the Redis Pod
```shell
kubectl delete pod redis
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

2. Run the following command to enter the Redis Pod and to check the current configuration:
```shell
kubectl exec -it redis -- redis-cli
```

3. Run the following command to to confirm that your Redis `maxmemory` value is set to the updated value of `2097152`:
```shell
127.0.0.1:6379> CONFIG GET maxmemory
```

4. Run the following command to confirm that your Redis `maxmemory-policy` value is set to the updated value of `allkeys-lru`:
```shell
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

5. Run the followin command to delete the created resources and clean up:
```shell
kubectl delete pod/redis configmap/example-redis-config
```
---

## Next steps
* Learn more about [ConfigMaps](/docs/tasks/configure-pod-container/configure-pod-configmap/).
* Learn how to [update configuration with a ConfigMap](/docs/tutorials/configuration/updating-configuration-via-a-configmap/).
