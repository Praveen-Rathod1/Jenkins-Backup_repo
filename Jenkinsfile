pipeline {
    agent any

    parameters {
        string(
            name: 'SERVER_IP',
            description: 'Tomcat Server IP'
        )
    }

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SERVER_USER='ubuntu'
        TOMCAT_DIR='/opt/tomcat/webapps'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Praveen-Rathod1/Jenkins-Backup_repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'

                script {

                    env.WAR_FILE = sh(
                    script: "find . -name '*.war' | head -1",
                    returnStdout: true
                    ).trim()

                    env.WAR_NAME = sh(
                    script: "basename ${env.WAR_FILE}",
                    returnStdout: true
                    ).trim()

                    echo "WAR File=${env.WAR_FILE}"
                }
            }
        }

        stage('Deploy') {
            steps {

                sshagent(['tomcat-ssh-key']) {

                    sh """
                    scp -o StrictHostKeyChecking=no \
                    ${WAR_FILE} ${SERVER_USER}@${params.SERVER_IP}:/tmp/

                    ssh -o StrictHostKeyChecking=no \
                    ${SERVER_USER}@${params.SERVER_IP} "
                    sudo rm -rf ${TOMCAT_DIR}/sample-app*
                    sudo mv /tmp/${WAR_NAME} ${TOMCAT_DIR}/
                    sudo systemctl restart tomcat
                    "
                    """
                }
            }
        }
    }
}
