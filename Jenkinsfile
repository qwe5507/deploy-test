pipeline {
    agent any

    tools {
        maven "M3"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/qwe5507/deploy-test', branch: 'main', credentialsId: 'github_token'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    sh 'mvn clean package'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    def jarFile = 'target/shortenurlservice-0.0.1-SNAPSHOT.jar'
                    def serverIp = '158.247.255.116'
                    def deployPath = '/root'
                    def runAppCommand = "nohup java -jar $deployPath/shortenurlservice-0.0.1-SNAPSHOT.jar > $deployPath/app.log 2>&1 &"
                    def checkLogCommand = "grep -q 'Started ShortenurlserviceApplication in' $deployPath/app.log"
                    
                    // 서버에 파일을 SCP로 전송
                    sh "scp -o StrictHostKeyChecking=no $jarFile root@$serverIp:$deployPath/"
                    
                    // 원격 서버에서 애플리케이션 비동기 실행
                    sshagent(['deploy_ssh_key']) {
                        sh "ssh -o StrictHostKeyChecking=no root@$serverIp '$runAppCommand'"
                        sleep 20 // 애플리케이션이 시작될 시간을 제공합니다.
                        
                        // 로그 파일을 확인하여 애플리케이션 실행 확인
                        int result = sh script: "ssh -o StrictHostKeyChecking=no root@$serverIp '$checkLogCommand'", returnStatus: true
                        if (result == 0) {
                            echo 'Deployment was successful.'
                        } else {
                            error 'Deployment failed.'
                        }
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'Deployment encountered an error.'
        }
    }
}
