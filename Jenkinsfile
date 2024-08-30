pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16' 
    }

    environment {
        GIT_REPO_URL = 'https://github.com/ravithikane02/swiggy_clone_k8s.git'
        BRANCH = 'main'
        SCANNER_HOME = tool name: 'SonarQubeScanner'
        SONARQUBE_SERVER = 'SonarQube_server'
    }

    stages {
        stage('Clear workspace') {
            steps {
                cleanWs()
            }
        }

        stage('GIT Checkout') {
            steps {
                git url: env.GIT_REPO_URL, branch: env.BRANCH
                echo 'Code Cloned'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    // Run the SonarQube Scanner
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Swiggy-CICD \
                        -Dsonar.projectKey=Swiggy-CICD'''
                }
            }
        }
        stage('Quality gates') {
            steps {
                // timeout(time: 1, unit: 'MINUTES') {
                //     waitForQualityGate abortPipeline: true
                // }
                waitForQualityGate abortPipeline: true
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        stage('Docker build and Push'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerHub', toolName: 'docker'){
                        echo 'Building Project....'
                        sh 'docker build -t swiggy-clone:latest .'
                        echo 'Push to DockerHub'
                        sh 'docker tag swiggy-clone:latest ravithikane02/swiggy-clone:latest'
                        sh 'docker push ravithikane02/swiggy-clone:latest'
                    }
                }
                
            }
        }
        stage('Trivy'){
            steps{
                sh 'trivy image ravithikane02/swiggy-clone:latest > trivyimage.txt'
            }
        }
        stage('Clear workspace at last') {
            steps {
                cleanWs()
            }
        }
    }
}
