# E5-VIRTDC
DE STEPHANIS Antonio Rendu




<!-- PROJECT LOGO --> <br /> <h3 align="center">Projet E5-VIRTDC:Stripe</h3> 



<!-- ABOUT THE PROJECT -->
## Fichier Manifest Rocket App et Stripe App
. Contenu du Manifest Rocket App


``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: rocket-configmap
  namespace: rocket-eval
data:
  DEMO_MODE: "True"
  DEBUG: "True"
---
apiVersion: v1
kind: Secret
metadata:
  name: stripe-secret
  namespace: rocket-eval
type: Opaque
# Pour mettre les secrets en claires, il faut utiliser "stringData:", sinon, il va falloir les hasher en base64
stringData:
#data:
  STRIPE_PUBLISHABLE_KEY: pk_test_51QkO2919ekhe451nCt7tL1SUvt4eozTsuGWl4jz8kCEDJmWDFpHJy63SQB2AjH8UyyeMU4mLuFDQPTZjXQUNbK9e00wvYq2O6A
  STRIPE_SECRET_KEY: sk_test_51QkO2919ekhe451n0XARU0afUhoQvPJxf6o7IxP3TU4VZJhDCHJ7j4te3T5wEcom6hSZFTZyqdjgwkI5LiQldzYT007PMAoTJV
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rocket-deployment
  namespace: rocket-eval
  # Labels du deployment
  labels:
    app: django
    env: preprod
    tier: frontend
spec:
  selector:
    matchLabels:
      app: django
  replicas: 1
  template:
    metadata:
      # Labels des pods
      labels:
        app: django
        env: preprod
        tier: frontend
    spec:
      containers:
      - name: rocket-random
        image: toniocs/rocket-app:prod
        resources:
          requests:
            memory: 64Mi
            cpu: 250m
          limits:
            memory: 120Mi
            cpu: 300m
        envFrom:
        - configMapRef:
            # nom de la configMap
            name: rocket-configmap
        env:
          # nom de la variable qui sera injectée
          - name: STRIPE_PUBLISHABLE_KEY
            valueFrom:
              secretKeyRef:
                # nom de l'objet secret
                name: stripe-secret
                # nom du secret précis
                key: STRIPE_PUBLISHABLE_KEY
          - name: STRIPE_SECRET_KEY
            valueFrom:
              secretKeyRef:
                name: stripe-secret
                key: STRIPE_SECRET_KEY
---
apiVersion: v1
kind: Service
metadata:
  # name: django-new-service
  name: rocket-service
  namespace: rocket-eval
spec:
  type: LoadBalancer
  selector:
    app: django
  ports:
  - port: 5005
    targetPort: 5005
    # nodePort: 30001
    name: django-np
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-rocketapp
  namespace: rocket-eval
spec:
  defaultBackend:
    service:
      name: rocket-service
      port:
        number: 7777
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: rocket-hpa
  namespace: rocket-eval
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: rocket-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

. Contenu du Manifest Stripe App

``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: stripe-flask-app
  labels:
    app: stripe-flask-app
spec:
  selector:
    matchLabels:
      app: stripe-flask-app
  template:
    metadata:
      labels:
        app: stripe-flask-app
    spec:
      containers:      
      - name: stripe-flask-app 
        image: stripe-app:latest
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 4242
        resources:
          requests:
            memory: 64Mi
            cpu: 100m
          limits:
            memory: 128Mi
            cpu: 200m
        env:
          # nom de la variable qui sera injectée
          - name: STRIPE_PUBLISHABLE_KEY
            valueFrom:
              secretKeyRef:
                # nom de l'objet secret
                name: stripe-secret-app
                # nom du secret précis
                key: STRIPE_PUBLISHABLE_KEY
          - name: STRIPE_SECRET_KEY
            valueFrom:
              secretKeyRef:
                name: stripe-secret-app
                key: STRIPE_SECRET_KEY
---
apiVersion: v1
kind: Service
metadata:
  name: stripe-flask-service
spec:
  type: NodePort
  selector:
    app: stripe-flask-app
  ports:
  - port: 4242
    protocol: TCP
    targetPort: 4242
---
apiVersion: v1
kind: Secret
metadata:
  name: stripe-secret-app
type: Opaque
stringData:
  STRIPE_PUBLISHABLE_KEY: pk_test_51QkO2919ekhe451nCt7tL1SUvt4eozTsuGWl4jz8kCEDJmWDFpHJy63SQB2AjH8UyyeMU4mLuFDQPTZjXQUNbK9e00wvYq2O6A
  STRIPE_SECRET_KEY: sk_test_51QkO2919ekhe451n0XARU0afUhoQvPJxf6o7IxP3TU4VZJhDCHJ7j4te3T5wEcom6hSZFTZyqdjgwkI5LiQldzYT007PMAoTJV
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: stripe-checkout-ingress
spec:
  defaultBackend:
    service:
      name: stripe-flask-service
      port:
        number: 7777
```


<!-- GETTING STARTED -->

### Pour déployer les applications:

1- Exécuter la commande suivante pour mettre en place vos application avec le manifest:

``` bash
k apply -f "nom du fichier.yaml" -n "nom du namespace"
```

2- Et vous pouvez voir vos application déployer en faisant:

``` bash
k get all
```


. Docker Hub repositories

![alt text](DockerHub.png)

<p align="right">(<a href="#readme-top">back to top</a>)</p>
