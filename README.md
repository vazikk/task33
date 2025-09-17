# task33

Сначала запустил K8S, установил helm и ArgoCD <br>
`curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash` <br>

`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml` <br>

Потом создал шаблом в helm: <br>

`helm create nginx-app` <br>

Записал переменные в values.yaml <br>
```
replicaCount: 1

image:
  repository: nginx
  tag: "v1"

service:
  type: NodePort
  port: 80
  targetPort: 80
  nodePort: 30000

resources: {}
```
<br>

Потом написал файл deployment.yaml в templates <br>
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  labels:
    app: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      serviceAccountName: {{ include "nginx-app.serviceAccountName" . }}
      containers:
        - name: nginx
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.targetPort }}
```
<br>

И файл service.yaml <br>
```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
      nodePort: {{ .Values.service.nodePort }}
  selector:
    app: {{ .Release.Name }}
```
<br>

Закинул это всё в Git: <br>

<img width="1052" height="297" alt="image" src="https://github.com/user-attachments/assets/8a6e80ac-8c03-40c3-b73a-3ae0a4f8bed1" /> <br>

<br>

Ну и собственно написал файл Application: <br>
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: git@github.com:vazikk/FOR33.git
    targetRevision: HEAD
    path: nginx-app
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Запустил: <br>
<img width="974" height="401" alt="image" src="https://github.com/user-attachments/assets/5cfe7c2f-e052-4845-b6c2-af41d2885e24" /> <br>

Итог: <br>

<img width="1237" height="431" alt="image" src="https://github.com/user-attachments/assets/7e4e9a22-32d7-4a27-9de5-0619846af5b5" />










