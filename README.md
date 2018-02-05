 Redis Master-Slave (Replication)

## Overview

This K8S module creates Redis Master-Slave deployment in Kubernetes

## Creating Redis

1. The first step is to create persistent volume claim needed for persistence
```

$ kubectl create persistenvolumeclaim

```

[Kubernetes PersistentVolumeClaim documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

To change storage settings you must modify files in the persistentvolumeclaim. You can change for example access modes, resources(size) and so forth.

2. Next step is to create configmaps.
```

$ kubectl create configmap

```

[Kubernetes ConfigMap documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
[Kubernetes Redis ConfigMap documentation](https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/)

If you want to change Redis configuration (redis.conf file), you have to edit files in the confimap directory.
For example you want to modify port Redis listen for Redis-Master. You must modify configmap/redis-master.yaml. New port will be 6370, so your configuration should look as follows:

```

apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-master
data:
  redis.conf: |
    appendonly yes
    port 6379

```

3. Create Redis services
```

$ kubectl create services

```

[Kubernetes Services documentation](https://kubernetes.io/docs/concepts/services-networking/service/)

4. Create Redis containers master and slave
```

$ kubectl create deployment

```

[Kubernetes deployment documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

Deployment files instructs Kubernetes how to create instances of your applicaiton. How many pods you want to run, mounted volumes and so forth.

```

apiVersion: extensions/v1beta1
**kind: Deployment**
metadata:
  name: "redis-master"
spec:
  **replicas: 1** > Define how many pods you want to run
  template:
    metadata:
      labels:
        app: redis
        name: "redis-master"
    spec:
      containers:
        - name: redis
          **image: launcher.gcr.io/google/redis4** > Define images which will be used to create instances
          args:
            - /etc/redis/redis.conf
          resources:
            requests:
              cpu: "100m"
              memory: "100Mi"
          ports:
            - name: redis
              **containerPort: 6379** > Define container port
              protocol: "TCP"
          **volumeMounts:** > Define volume mounts
            - name: "redisconfig"
              mountPath: "/etc/redis"
            - name: "redis-data"
              mountPath: "/var/lib/redis"
      volumes:
        - name: "redis-data"
          persistentVolumeClaim:
            claimName: redisdata-master
        - name: "redisconfig"
          configMap:
            name: "redis-master"
            items:
              - key: "redis.conf"
                path: "redis.conf"

```
