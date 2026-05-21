pipeline {
    agent any

    parameters {
        string(
            name: 'SERVER_IP',
            defaultValue: '',
            description: 'Tomcat Server IP'
        )
    }

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SERVER_USER = 'ubuntu'
        TOMCAT_DIR = '/opt/tomcat/webapps'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Praveen-Rathod1/Jenkins-Backup_repo.git'
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                echo "Current workspace:"
                pwd

                echo "Repository structure:"
                ls -R
                '''
            }
        }

        stage('Build') {
            steps {

                dir('sample-app') {
                    sh 'mvn clean package -DskipTests'
                }

                script {

                    env.WAR_FILE = sh(
                        script: "find sample-app/target -name '*.war' | head -1",
                        returnStdout: true
                    ).trim()

                    env.WAR_NAME = sh(
                        script: "basename ${env.WAR_FILE}",
                        returnStdout: true
                    ).trim()

                    echo "WAR File: ${env.WAR_FILE}"
                    echo "WAR Name: ${env.WAR_NAME}"
                }
            }
        }

        stage('Upload to JFrog') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'jfrog-creds',
                        usernameVariable: 'JFROG_USER',
                        passwordVariable: 'JFROG_PASS'
                    )
                ]) {

                    sh '''
                    set -e

                    FILE_NAME="${JOB_NAME}-${BUILD_NUMBER}.war"

                    echo "Uploading to JFrog..."

                    curl --fail -L \
                    -u $JFROG_USER:$JFROG_PASS \
                    -T "$WAR_FILE" \
                    "https://triallb6d6m.jfrog.io/artifactory/jenkinsjava-generic-local/$FILE_NAME"

                    echo "Upload successful"
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {

                sshagent(credentials: ['tomcat-ssh-key']) {

                    sh """
                    set -e

                    echo "Copying WAR to target server..."

                    scp -o StrictHostKeyChecking=no \
                    ${WAR_FILE} \
                    ${SERVER_USER}@${params.SERVER_IP}:/tmp/

                    ssh -o StrictHostKeyChecking=no \
                    ${SERVER_USER}@${params.SERVER_IP} "

                    sudo rm -rf ${TOMCAT_DIR}/sample-app*
                    sudo mv /tmp/${WAR_NAME} ${TOMCAT_DIR}/

                    sudo systemctl restart tomcat

                    sleep 10

                    sudo systemctl status tomcat --no-pager
                    "

                    echo "Deployment completed successfully"
                    """
                }
            }
        }
    }

    post {

        success {
            echo 'Pipeline executed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }

        always {
            cleanWs()
        }
    }
}
