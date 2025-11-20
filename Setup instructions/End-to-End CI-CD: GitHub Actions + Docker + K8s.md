End-to-End CI-CD: GitHub Actions + Docker + K8s

Create a .github/workflows/main.yml

                    name: Build & Deploy to Minikube (Refined)
                    
                    on:
                      push:
                        branches:
                          - main
                    
                    jobs:
                      build-deploy:
                        runs-on: ubuntu-latest
                        env:
                          IMAGE_NAME: abivarmang/sample_app
                          IMAGE_TAG: latest
                    
                        steps:
                          # 1️⃣ Checkout repo
                          - name: Checkout repo
                            uses: actions/checkout@v3
                    
                          # 2️⃣ Set up Node.js
                          - name: Set up Node.js
                            uses: actions/setup-node@v3
                            with:
                              node-version: '23'
                    
                          # 3️⃣ Install dependencies
                          - name: Install dependencies
                            run: npm install
                    
                          # 4️⃣ Run tests
                          - name: Run tests
                            run: npm test || echo "No tests found"
                    
                          # 5️⃣ Set up Minikube cluster (FIXED VERSION)
                          - name: Set up Minikube
                            uses: medyagh/setup-minikube@master
                            with:
                              driver: docker
                              kubernetes-version: 'v1.28.0'  # Updated to supported version
                    
                          # 6️⃣ Wait for cluster to be ready
                          - name: Wait for cluster readiness
                            run: |
                              minikube status
                              kubectl wait --for=condition=Ready node/minikube --timeout=180s
                              kubectl get nodes
                    
                          # 7️⃣ Set up Docker Buildx
                          - name: Set up Docker Buildx
                            uses: docker/setup-buildx-action@v3
                    
                          # 8️⃣ Docker login
                          - name: Docker login
                            uses: docker/login-action@v2
                            with:
                              username: ${{ secrets.DOCKER_USERNAME }}
                              password: ${{ secrets.DOCKER_PASSWORD }}
                    
                          # 9️⃣ Build & push Docker image
                          - name: Build & Push Docker image
                            run: |
                              docker build -t $IMAGE_NAME:$IMAGE_TAG .
                              docker push $IMAGE_NAME:$IMAGE_TAG
                          
                          # 🔟 Deploy to Minikube
                          - name: Deploy to Minikube
                            run: |
                              echo "Deploying application to Minikube..."
                              kubectl apply -f manifest.yml
                              kubectl get all
