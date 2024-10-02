
# 📦 SpringApp Kubernetes 배포

Minikube를 사용하여 SpringApp을 데이터베이스 연동 없이 Kubernetes에 배포하는 방법을 설명합니다.

## 🎯 개요
- **데이터베이스 연동 없음**: 이 애플리케이션은 데이터베이스와 연결되지 않습니다.
- **컨테이너 요구사항**: Docker 이미지를 사용하여 컨테이너를 생성해야 합니다.
- **Pods**: 3개의 컨테이너(Pod)가 생성되어야 합니다.
- **외부 통신**: 외부와 통신할 수 있는 서비스 1개가 필요합니다.

---

## 📑 구성 아키텍처

아래 다이어그램은 Kubernetes 클러스터 내에서 **SpringApp**이 어떻게 배포되고 서비스되는지를 보여줍니다:

```bash
from diagrams import Cluster, Diagram
from diagrams.k8s.compute import Pod, Deployment, ReplicaSet
from diagrams.k8s.network import Service
from diagrams.onprem.client import Users
from diagrams.aws.network import ELB

with Diagram("SpringApp Kubernetes Architecture", show=False, direction="LR"):
    user = Users("User")

    with Cluster("Kubernetes Cluster"):
        lb_service = Service("springapp-service
(LoadBalancer)")

        with Cluster("SpringApp Deployment"):
            deploy = Deployment("springapp-deployment")

            with Cluster("ReplicaSet"):
                replica_set = ReplicaSet("springapp-replicaset")
                pods = [Pod("springapp-deployment-pod1"),
                        Pod("springapp-deployment-pod2"),
                        Pod("springapp-deployment-pod3")]

        user >> lb_service >> replica_set >> pods
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/6e7165f6-9e8f-4105-b3be-b9494dbf25e2" width="400"/>
</p>




## 🚀 1. Docker 이미지 빌드

1. **Dockerfile 작성**

   프로젝트 디렉토리에 `Dockerfile`을 생성하고 아래 내용을 작성하세요:

   ```dockerfile
   FROM openjdk:17-slim
   WORKDIR /app
   COPY SpringApp-0.0.1-SNAPSHOT.jar app.jar
   ENTRYPOINT ["java", "-jar", "app.jar"]
   ```

2. **Docker 이미지 빌드**

   아래 명령어로 Docker 이미지를 빌드합니다:

   ```bash
   docker build -t springapp .
   ```

3. **Docker Hub 로그인**

   Docker Hub에 이미지를 푸시하기 전에 로그인합니다:

   ```bash
   docker login
   ```

4. **Docker 이미지 태그 설정**

   Docker 이미지를 태그합니다:

   ```bash
   docker tag springapp cshharry/springapp:latest
   ```

5. **Docker Hub에 이미지 푸시**

   Docker Hub에 이미지를 푸시합니다:

   ```bash
   docker push cshharry/springapp:latest
   ```

---

## ⚙️ 2. Kubernetes에 Deployment 및 Service 설정

1. **Deployment YAML 파일 작성**

   3개의 Pod을 배포하기 위해 `springapp-deployment.yml` 파일을 작성합니다:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: springapp-deployment
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: springapp
     template:
       metadata:
         labels:
           app: springapp
       spec:
         containers:
         - name: springapp
           image: cshharry/springapp:latest
           ports:
           - containerPort: 8080
   ```

2. **Service YAML 파일 작성**

   외부 통신을 가능하게 하기 위해 `springapp-service.yml` 파일을 작성합니다:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: springapp-service
   spec:
     type: LoadBalancer
     selector:
       app: springapp
     ports:
     - protocol: TCP
       port: 80
       targetPort: 8080
   ```

3. **Deployment 및 Service 적용**

   아래 명령어로 Kubernetes에 설정을 적용합니다:

   ```bash
   kubectl apply -f springapp-deployment.yml
   kubectl apply -f springapp-service.yml
   ```

---

## ✅ 3. 서비스 상태 확인 및 접근

1. **Pod 및 서비스 상태 확인**

   아래 명령어로 Pod과 서비스가 정상적으로 실행되고 있는지 확인합니다:

   ```bash
   kubectl get pods
   kubectl get services
   ```
    출력:
    ```
    NAME                TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
    kubernetes          ClusterIP      10.96.0.1       <none>          443/TCP        128m
    springapp-service   LoadBalancer   10.102.16.209   10.102.16.209   80:31145/TCP   34m
    ```

위 출력에서 `springapp-service`의 **EXTERNAL-IP**는 `10.102.16.209`로 할당되었습니다. 이 IP를 통해 서비스에 외부에서 접속할 수 있습니다.

---
