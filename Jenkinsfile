pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'M3'
    }

    environment {
        DOCKERHUB_CRED = credentials('dockerCredentials')
    }

    stages {
        // GitHub에서 소스 코드 다운로드
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/choch4814/spring-petclinic.git',
                    branch: 'main'
            }
        }

        // Maven 빌드
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }

        // Docker 이미지 생성
        stage('Docker Image Create') {
            steps {
                echo 'Docker Image Create'
                sh '''
                    docker build -t spring-petclinic:${BUILD_NUMBER} .
                    docker tag spring-petclinic:${BUILD_NUMBER} choch4814/spring-petclinic:latest
                '''
            }
        }

        // Docker Hub 업로드
        stage('Docker Image Push') {
            steps {
                echo 'Docker Image Push'
                sh 'echo "$DOCKERHUB_CRED_PSW" | docker login -u "$DOCKERHUB_CRED_USR" --password-stdin'
                sh 'docker image push choch4814/spring-petclinic:latest'
            }
            post {
                always {
                    sh '''
                        docker rmi -f spring-petclinic:${BUILD_NUMBER} || true
                        docker rmi -f choch4814/spring-petclinic:latest || true
                        docker logout || true
                    '''
                }
            }
        }

        // Kubernetes에 WAS 배포
        stage('Kubernetes Deploy') {
            steps {
                echo 'Kubernetes Deploy'
                sh '''
                    kubectl apply -f k8s/spring-petclinic-deployment.yaml
                    kubectl rollout status deployment/spring-petclinic --timeout=180s
                '''
            }
        }
    }
}
