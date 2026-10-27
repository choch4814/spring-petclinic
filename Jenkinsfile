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
        // Github로부터 소스코드 다운로드
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/choch4814/spring-petclinic.git',
                branch: 'main'
            }
        }
        //Maven을 이요한 Build
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }
        // Docker image 생성
        stage ('Docker Image Create') {
            steps {
            echo 'Docker Image Create'
            sh '''
                docker build -t spring-petclinic:${BUILD_NUMBER} .
                docker tag spring-petclinic:${BUILD_NUMBER} choch4814/spring-petclinic:latest
                '''
            }
        }
        // dockerhub image Push
        stage('Docker Image Push') {
            steps {
                echo 'docker Image Push'
                // Docker Login
                sh 'echo ${DOCKERHUB_CRED_PSW} | docker login -u ${DOCKERHUB_CRED_USR} --password-stdin'

                // Docker Hub Push
                sh 'docker image push choch4814/spring-petclinic:latest'
            }
            post {
                always {
                    sh '''
                    docker rmi -f spring-petclinic:${BUILD_NUMBER}
                    docker rmi -f choch4814/spring-petclinic:latest
                    '''
                }
            }
        }
        // docker Image 삭제
        // SSH를 이용한 Docker 컨테이너 실행
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
                execTimeout: 120000, 
                flatten: false, 
                makeEmptyDirs: false, 
                noDefaultExcludes: false, 
                patternSeparator: '[, ]+', 
                remoteDirectory: '', 
                remoteDirectorySDF: false, 
                removePrefix: '', 
                sourceFiles: 'target/*.jar')],
                 usePromotionTimestamp: false, 
                useWorkspaceInPromotion: false, 
                verbose: false)])
            }
        }                
    }
}
