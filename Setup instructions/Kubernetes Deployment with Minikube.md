Kubernetes Deployment with Minikube

Install Minikube & kubectl:
    
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    sudo install kubectl /usr/local/bin/kubectl
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    minikube start

 manifest.yml     

 # Deployment         
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: sample-app
            spec:
              replicas: 2
              selector:
                matchLabels:
                  app: sample-app
              strategy:
                type: RollingUpdate
              template:
                metadata:
                  labels:
                    app: sample-app
                spec:
                  containers:
                    - name: sample-app
                      image: abivarmang/sample_app:latest
                      ports:
                        - containerPort: 3000
            
            ---
            
 # Service           
            apiVersion: v1
            kind: Service
            metadata:
              name: sample-app
            spec:
              type: ClusterIP
              selector:
                app: sample-app
              ports:
                - port: 80
                  targetPort: 3000
            
            ---
            
  # Ingress          
            apiVersion: networking.k8s.io/v1
            kind: Ingress
            metadata:
              name: sample-app-ingress
              annotations:
                nginx.ingress.kubernetes.io/rewrite-target: /
            spec:
              rules:
                - host: sample.local
                  http:
                    paths:
                      - path: /
                        pathType: Prefix
                        backend:
                          service:
                            name: sample-app
                            port:
                              number: 80

Deploy & Verify:

                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                kubectl get pods
                kubectl get svc
                minikube service sample-app-service

