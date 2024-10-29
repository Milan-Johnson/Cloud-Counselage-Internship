
pipeline{
    agent any

    tools{
        maven 'Maven3'
    }

    environment {
        APP_NAME = "CCI-Project"
        RELEASE = "1.0.0"
        DOCKER_USER = "milanjohnson"
        DOCKER_PASS = 'Docker-CCI'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"

    }
  
    
    stages {
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }
        }
        stage('Checkout from SCM'){
            steps {
                git(url: 'https://github.com/Milan-Johnson/Cloud-Counselage-Internship', branch: 'main')
            }
        }
        stage('Build Application'){
            steps {
                sh "mvn clean package"
            }
        }
        stage('Test Application'){
            steps {
                sh "mvn test"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'cci-aws') {
                    sh '''mvn clean package sonar:sonar''' 
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                    
                }
            }
        }
        stage("Docker build") {
            steps{
                sh "sudo docker version"
                sh "sudo docker build -t ${IMAGE_NAME} ."
                sh "docker image list"
                sh "docker tag ${IMAGE_NAME} ${IMAGE_TAG}:latest"
            }
        }
        stage("Docker push") {
            steps{
                withCredentials([string(credentialsId: 'cci-docker-token', variable: 'DOCKER_PSD')]){
                    sh "sudo docker login -u ${DOCKER_USER} -p ${DOCKER_PSD}"
                }
                sh "sudo docker push ${IMAGE_NAME}:${IMAGE_TAG} "
                sh "sudo docker push ${IMAGE_NAME}:latest "                
            }
        }
        stage("Kubernetes Deployment") {
            steps{
                sh 'kubectl apply -f deployment.yml'
                sh 'kubectl set image deployment/cci-springboot springboot=milanjohnson/cci-project:${IMAGE_TAG}'
            }
        }
    }
}
