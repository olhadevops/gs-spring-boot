pipeline {
    agent any

    // 2. Інструменти, які потрібно підготувати
    tools {
        maven 'Maven 3.9.10'
    }

    // 3. Етапи виконання пайплайну
    stages {
        // Етап 1: Збірка проєкту
        stage('Build') {
            steps {
                sh 'cd complete && mvn clean install'
            }
        }

        // Етап 2: Розгортання на сервері EC2
        stage('Deploy to EC2') {
            steps {
                withCredentials([string(credentialsId: 'ec2-server-ip', variable: 'SERVER_IP')]) {
                    sshagent(credentials: ['ec2-ssh-key']) {
                        sh """
                            # 1. Копіюємо зібраний .jar файл на сервер
                            scp -o StrictHostKeyChecking=no complete/target/*.jar ubuntu@${SERVER_IP}:~/app/
                            
                            # 2. Виконуємо команди на віддаленому сервері як єдиний рядок
                            ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "pkill -f 'spring-boot-complete' || echo 'No process to kill'; cd ~/app && nohup java -jar *.jar > app.log 2>&1 </dev/null &"
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished with status: ${currentBuild.currentResult}"
        }
    }
}
