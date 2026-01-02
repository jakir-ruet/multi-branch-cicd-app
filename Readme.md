## Production Grade Multi Branch CI/CD Pipeline App

### Run the Flask App

```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install flask
python app.py
```

#### How to build, push & pull from Docker Hub

```bash
docker build -t jakirbd/multi-branch-cicd-app:latest .
docker tag existing-image-id jakirbd/multi-branch-cicd-app:latest # If already built
docker push jakirbd/multi-branch-cicd-app:latest
docker pull jakirbd/multi-branch-cicd-app:latest
```

> `Kubernetes` & `ArgoCD` files exist only `Master` Branch.
