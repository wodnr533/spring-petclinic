pipeline {
    agent any
    
    tools {
        jdk 'JDK17'
        maven 'M3'
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('Docker-token')
    }

    stages {

        stage('Git Clone') {
            steps {
                git url: 'https://github.com/wodnr533/spring-petclinic.git',
                    branch: 'main'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                docker build -t wodnr8174/spring-petclinic:${BUILD_NUMBER} .
                docker tag wodnr8174/spring-petclinic:${BUILD_NUMBER} wodnr8174/spring-petclinic:latest
                """
            }
        }

        stage('Docker Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Docker Push') {
            steps {
                sh 'docker push wodnr8174/spring-petclinic:latest'
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl apply -f k8s/'
                    sh 'kubectl set image deployment/petclinic petclinic=wodnr8174/spring-petclinic:latest'
                }
            }
        }
    }
}
