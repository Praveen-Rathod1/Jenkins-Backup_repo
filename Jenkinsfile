stage('Deploy to Tomcat') {
    steps {
        sshagent(['ubuntu']) {
            sh '''
                set -e

                echo "Deploying WAR..."

                WAR_FILE=$(ls sample-app/target/*.war)

                EC2_HOST="YOUR_EC2_PUBLIC_IP"

                scp -o StrictHostKeyChecking=no $WAR_FILE ubuntu@$EC2_HOST:/tmp/

                ssh -o StrictHostKeyChecking=no ubuntu@$EC2_HOST "
                    sudo mv /tmp/*.war /opt/tomcat/webapps/
                    sudo systemctl restart tomcat
                "
            '''
        }
    }
}
