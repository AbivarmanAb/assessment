# assessment
CI/CD Pipeline with Jenkins + Docker

Objective: Build, test, and run a Dockerized app using Jenkins.

Prepare GitHub Repo
           Create a GitHub repository for your sample app (Node.js or Python).
           Add your app code and a Dockerfile.
           Example Node.js Dockerfile:

            FROM  node:18.9.1
            WORKDIR /app
            COPY package*.json ./
            RUN npm install
            COPY . .
            EXPOSE 3000
            CMD ["npm","start"]

Run Jenkins via Docker

 docker run -d --name jenkins -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts


Set Up Jenkins Pipeline

    Go to http://localhost:8080, unlock Jenkins, install recommended plugins.

    Add GitHub credentials in Jenkins (username & token).

    Create a Pipeline Job and point it to your GitHub repo.



    pipeline {
        agent any
    
        // Set up tools automatically from Jenkins global configuration
        tools {
            jdk 'jdk'         // Name in Jenkins Global Tool Config
            nodejs 'node js'  // Name in Jenkins Global Tool Config
        }
    
        // Environment variables
        environment {
            DOCKER_HUB_PASSWORD = credentials('Docker_Password')
        }
    
        stages {
            stage('Check Versions') {
                steps {
                    sh 'java -version'
                    sh 'node -v'
                }
            }
    
            stage('Clean Workspace') {
                steps {
                    cleanWs()
                }
            }
    
            stage('Checkout from Git') {
                steps {
                    git branch: 'main',
                        credentialsId: 'github',
                        url: 'https://github.com/AbivarmanAb/assessment.git'
                }
            }
    
            stage('Install Dependencies') {
                steps {
                    sh 'npm install'
                }
            }
    
            stage('Docker Build & Push') {
                steps {
                    script {
                        // Docker login
                        sh "docker login -u abivarmang -p $DOCKER_HUB_PASSWORD"
    
                        // Build Docker image
                        sh 'docker build -t sample_app .'
    
                        // Tag image for Docker Hub
                        sh 'docker tag sample_app abivarmang/sample_app:latest'
    
                        // Push to Docker Hub
                        sh 'docker push abivarmang/sample_app:latest'
                    }
                }
            }
    
            stage('Deploy to Container') {
                steps {
                    // Remove existing container if running
                    sh 'docker rm -f sample_app || true'
    
                    // Run new container
                    sh 'docker run -d --name sample_app -p 3000:3000 abivarmang/sample_app:latest'
                }
            }
        }
    
        post {
            always {
                emailext(
                    attachLog: true,
                    subject: "Build ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                    <html>
                    <body>
                        <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                        </div>
                        <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                        </div>
                        <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        </div>
                    </body>
                    </html>
                    """,
                    to: 'abivarman5032@gmail.com',
                    mimeType: 'text/html'
                )
            }
        }
    }


    
