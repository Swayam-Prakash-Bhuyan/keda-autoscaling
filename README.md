
---

# 🟦 **STEP 1 — Install KEDA (from scratch)**

```bash
kubectl apply --server-side -f https://github.com/kedacore/keda/releases/download/v2.14.0/keda-2.14.0.yaml
```

---

# 🟦 **STEP 2 — Install the HTTP Add-on**

```bash
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
helm install http-add-on kedacore/keda-add-ons-http --namespace keda
```

---

# 🟩 **STEP 3 — Create NGINX Deployment (cat EOF)**

```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  labels:
    app: nginx-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
        - name: nginx-app
          image: nginx:latest
          ports:
            - containerPort: 80
EOF
```

---

# 🟩 **STEP 4 — Create Service (NodePort)**

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx-app
spec:
  type: NodePort
  selector:
    app: nginx-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 31001
EOF
```

---

# 🟩 **STEP 5 — Create ScaledObject (Cron scaler)**

This scales NGINX based on CRON schedule (example).

```bash
cat <<EOF | kubectl apply -f -
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: nginx-deployment
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-app

  pollingInterval: 5
  cooldownPeriod: 300

  minReplicaCount: 0
  maxReplicaCount: 5

  fallback:
    failureThreshold: 3
    replicas: 1

  triggers:
  - type: cron
    metadata:
      timezone: Asia/Kolkata
      start: "30 * * * *"
      end: "45 * * * *"
      desiredReplicas: "5"
EOF
```

---

# 🟩 **STEP 6 — Create HTTPScaledObject (HTTP Autoscaling)**

This is the **correct** new CRD spec (NO deprecated fields).

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: http.keda.sh/v1alpha1
kind: HTTPScaledObject
metadata:
  name: nginx-app-http
  namespace: default
spec:
  hosts:
  - myhost.com
  scaleTargetRef:
    name: nginx-app
    service: nginx-app
    port: 80
  replicas:
    min: 0
    max: 10
EOF
```

---

# 🟦 **STEP 7 — Verify Everything**

```bash
kubectl get pods
kubectl get scaledobjects
kubectl get httpscaledobjects
kubectl get svc nginx-app
```

---

# 🟦 **STEP 8 — Test HTTP Autoscaling**

Send load (example):

```bash
kubectl port-forward svc/keda-add-ons-http-interceptor-proxy -n keda 32500:8080

for i in {1..2000}; do
  curl -s -o /dev/null -H "Host: myhost.com" http://localhost:32500 &
done
wait
```

Check replica count:

```bash
kubectl get deploy nginx-app -w
```

---


