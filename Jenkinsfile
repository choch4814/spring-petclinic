pipeline {
    agent any
    tools {
        jdk 'JDK21'
        maven 'M3'
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
        //SSH를 이용한 실행파일 전송
        stage('SSH Publish') {
            steps {
                echo 'SSH Publish'
                sshPublisher(publishers: [sshPublisherDesc(configName: 'target', 
                transfers: [sshTransfer(cleanRemote: false, excludes: '', 
                execCommand: '''
                docker rmi -f $(docker ps -aq)
                docker rmi -f $(docker images -q)
                docker run -itd -p 80:8080 --name=spring-petclinic choch4814/spring-petclinic:latest
                ''', 
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
