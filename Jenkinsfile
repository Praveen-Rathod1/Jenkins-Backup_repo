pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Praveen-Rathod1/Jenkins-Backup_repo.git'
            }
        }

        stage('Find Project') {
            steps {
                sh '''
                    echo "Searching for pom.xml..."
                    find . -name pom.xml
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    POM_PATH=$(find . -name pom.xml | head -n 1)

                    if [ -z "$POM_PATH" ]; then
                        echo "ERROR: No pom.xml found in repository"
                        exit 1
                    fi

                    echo "Using POM: $POM_PATH"

                    mvn clean package -DskipTests -f $POM_PATH
                '''
            }
        }

        stage('Upload to JFrog') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'jfrog-creds',
                    usernameVariable: 'JFROG_USER',
                    passwordVariable: 'JFROG_PASS'
                )]) {

                    sh '''
                        echo "Uploading WAR to JFrog..."

                        WAR_FILE=$(find . -name "*.war" | head -n 1)

                        if [ -z "$WAR_FILE" ]; then
                            echo "ERROR: WAR file not found"
                            exit 1
                        fi

                        echo "Found WAR: $WAR_FILE"

                        FILE_NAME="${JOB_NAME}-${BUILD_NUMBER}-sample.war"

                        curl -u $JFROG_USER:$JFROG_PASS -T "$WAR_FILE" \
                        "https://triallb6d6m.jfrog.io/artifactory/jenkinsjava-generic-local/$FILE_NAME"
                    '''
                }
            }
        }
    }
}
