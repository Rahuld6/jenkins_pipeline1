pipeline {
    agent any

    environment {
        TOMCAT_IP = '172.31.14.230'
        TOMCAT_USER = 'rahul'
        TOMCAT_PASS = credentials('tomcat-manager')
        APP_NAME = 'myapp'
        REPO = 'https://github.com/Rahuld6/jenkins_pipeline.git'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${REPO}"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent(['tomcat-ssh']) {
                    sh """
                        scp -o StrictHostKeyChecking=no target/${APP_NAME}.war ec2-user@${TOMCAT_IP}:/tmp/${APP_NAME}.war
                    """

                    // Deploy via Tomcat Manager App text interface
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${TOMCAT_IP} \\
                        "curl --upload-file /tmp/${APP_NAME}.war \\
                        --user ${TOMCAT_USER}:${TOMCAT_PASS} \\
                        http://localhost:8080/manager/text/deploy?path=/${APP_NAME}&update=true"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully! Access your app at http://${TOMCAT_IP}:8080/${APP_NAME}"
        }
        failure {
            echo "Deployment failed. Check build and Tomcat logs."
        }
    }
}

