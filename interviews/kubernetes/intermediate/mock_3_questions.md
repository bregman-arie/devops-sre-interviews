# Kubernetes - Intermediate Interview Mock #3

> **Difficulty:** Intermediate  
> **Duration:** ~45 minutes  
> **Goal:** Assess your understanding of Kubernetes storage, stateful applications, backup/recovery, and data persistence strategies.

---

## 🧠 Section 1: Core Questions

1. Explain the difference between **Static** and **Dynamic** volume provisioning. When would you use each? [📖 Answer](mock_3_answers.md#1-explain-the-difference-between-static-and-dynamic-volume-provisioning-when-would-you-use-each)
2. What are **StorageClasses** and how do they enable dynamic provisioning? [📖 Answer](mock_3_answers.md#2-what-are-storageclasses-and-how-do-they-enable-dynamic-provisioning)
3. How do **StatefulSets** differ from **Deployments** when managing stateful applications? [📖 Answer](mock_3_answers.md#3-how-do-statefulsets-differ-from-deployments-when-managing-stateful-applications)
4. Explain **Volume Access Modes** (ReadWriteOnce, ReadOnlyMany, ReadWriteMany) and their use cases. [📖 Answer](mock_3_answers.md#4-explain-volume-access-modes-readwriteonce-readonlymany-readwritemany-and-their-use-cases)
5. How do you implement **backup and disaster recovery** strategies for Kubernetes stateful workloads? [📖 Answer](mock_3_answers.md#5-how-do-you-implement-backup-and-disaster-recovery-strategies-for-kubernetes-stateful-workloads)
6. What are **Volume Snapshots** and how do they help with data management? [📖 Answer](mock_3_answers.md#6-what-are-volume-snapshots-and-how-do-they-help-with-data-management)
7. How do you handle **storage migration** and **volume expansion** in production clusters? [📖 Answer](mock_3_answers.md#7-how-do-you-handle-storage-migration-and-volume-expansion-in-production-clusters)
8. What are the challenges of running **databases in Kubernetes** and how do you address them? [📖 Answer](mock_3_answers.md#8-what-are-the-challenges-of-running-databases-in-kubernetes-and-how-do-you-address-them)

---

## ⚙️ Section 2: Scenario

**Scenario:**  
Your organization runs a critical e-commerce platform with the following stateful components:
- PostgreSQL primary/replica database cluster
- Redis for session storage and caching  
- Elasticsearch for search and analytics
- File storage for product images and documents

The platform requires:
- High availability with automated failover
- Point-in-time recovery capabilities
- Performance optimization for different workload types
- Compliance with data retention policies
- Cost-effective storage tiering

Design a comprehensive storage architecture that addresses scalability, reliability, and operational requirements.

[📖 Answer](mock_3_answers.md#️-section-2-scenario---answer)

---

## 🧩 Section 3: Problem-Solving

**Task:**  
A team deployed a MongoDB replica set using StatefulSets, but they're experiencing several issues with data persistence and performance. The application occasionally loses data after pod restarts, and read performance is poor during peak hours.

**Current Configuration:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
  namespace: database
spec:
  serviceName: mongodb-service
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:4.4
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: admin
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: password123
        volumeMounts:
        - name: mongodb-storage
          mountPath: /data/db
        resources:
          requests:
            memory: 1Gi
            cpu: 500m
          limits:
            memory: 2Gi
            cpu: 1000m
  volumeClaimTemplates:
  - metadata:
      name: mongodb-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi

---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  namespace: database
spec:
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017
  clusterIP: None

---
# Basic StorageClass being used
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  replication-type: none
reclaimPolicy: Delete
allowVolumeExpansion: false
```

**Reported Issues:**
1. Data loss after pod failures or restarts
2. Poor read performance during high load
3. No backup strategy in place  
4. Storage costs are high for the allocated capacity
5. Cannot scale storage when needed
6. MongoDB replica set is not properly configured
7. No monitoring for storage performance

Identify the problems and provide an improved configuration that ensures data durability, performance, and operational excellence.

[📖 Answer](mock_3_answers.md#-section-3-problem-solving---answer)
