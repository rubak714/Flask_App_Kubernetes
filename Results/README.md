### Final Successful Deployment Breakdown

Here’s what happened step-by-step to successfully deploy your Flask app to Kubernetes using Minikube:

---

### Step 1: Connect Docker to Minikube’s Docker Daemon
```bash
eval $(minikube docker-env)
``` 
This ensures any image you build using `docker build` is available **inside Minikube’s Docker environment**, not your host. This avoids `ImagePullBackOff` errors.

---

### Step 2: Build Docker Image
```bash
docker build -t flask-cicd-app .
``` 
The image `flask-cicd-app:latest` was built successfully **inside Minikube's Docker engine**.

---

### Step 3: Apply Deployment
```bash
kubectl apply -f k8s/deployment.yaml
```
This created a Deployment named `flask-app-deployment` using your custom Docker image.

**Fixed Markdown Block that helped:**
```yaml
image: flask-cicd-app:latest
```
This line matches the image you built **locally inside Minikube**, so Kubernetes could find and run it.

---

### Step 4: Verify Pod Status
```bash
kubectl get pods
```
**Output:**
```
NAME                                   READY   STATUS    RESTARTS   AGE
flask-app-deployment-d7fbc47bb-4mzml   1/1     Running   0          12s
```
This means the Pod is up and running, container is healthy, and probes passed.

---

### Step 5: Check Deployment Status
```bash
kubectl get deployments
```
**Output:**
```
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
flask-app-deployment   1/1     1            1           29s
```
Deployment is successful: one replica running and available.

---

### Step 6: Check Service
```bash
kubectl get svc
```
**Output:**
```
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
flask-service   NodePort    10.101.173.79   <none>        80:30007/TCP   46m
```
`flask-service` exposes your Flask app to outside Minikube on port `30007`. You can access it with:
```bash
minikube service flask-service
```

---

### Debugging: Deployment Configuration Update (Flask App on Minikube)

#### New Changes in `deployment.yaml`

The following updates were made to the `containers` section of your Kubernetes Deployment YAML to resolve the `ImagePullBackOff` error:

```yaml
containers:
- name: flask-container
  image: flask-cicd-app:latest
  imagePullPolicy: Never
  ports:
  - containerPort: 5000
```

#### Explanation of Each Change:

| Field             | Purpose                                                                                                                                                                                    |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `image`           | Specifies the image name: `flask-cicd-app:latest` (built locally).                                                                                                                         |
| `imagePullPolicy` | Set to `Never` to **force Kubernetes to use the local image** instead of trying to pull it from Docker Hub. This was crucial since the image only existed inside Minikube's Docker daemon. |
| `containerPort`   | Declares that the container listens on port `5000`, matching your Flask app.                                                                                                               |

#### Why This Solved the Problem

Before this change, Kubernetes tried to pull the image from Docker Hub, resulting in an `ImagePullBackOff` error. After updating:

* `eval $(minikube docker-env)` correctly redirected Docker to Minikube's internal environment.
* You rebuilt the Docker image inside Minikube.
* `imagePullPolicy: Never` told Kubernetes not to pull from a registry, avoiding the error.

This change ensured that the Pod could use the image directly from Minikube's Docker engine, resulting in a successfully running container.

---

Let me know if you'd like to expose this service publicly or push the image to a remote registry like DockerHub or GitHub Container Registry (GHCR).


### Summary of Changes That Fixed It
- Used `eval $(minikube docker-env)` before building.
- Built image **inside Minikube Docker daemon**.
- Correct `image:` name in `deployment.yaml` matched the one built.
- Probes and ports were correctly set.

now fully deployed and ready to access the app! 
