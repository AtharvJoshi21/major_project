pipeline {
    agent any
    parameters {
        string(name: 'PARAM_URL', defaultValue: '', description: 'The URL to be used')
        string(name: 'PARAM_EMAIL', defaultValue: '', description: 'The email address to be used')
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building Job....'
            }
        }
        stage('Test'){
            steps {
                echo 'Testing Job....'
            }
        }
        stage('Security Scan') {
            steps {
                script {
                    echo 'Running ZAP Security Scan...'
                    // Pull and run ZAP Docker container
                    sh """
                    docker run -u zap -d --name zap -p 8080:8080 owasp/zap2docker-stable zap.sh -daemon -port 8080 -host 0.0.0.0 -config api.disablekey=true
                    sleep 30  # Give ZAP some time to start
                    docker exec zap zap-cli quick-scan --self-contained --start-options '-config api.disablekey=true' ${params.PARAM_URL}
                    docker exec zap zap-cli report -o /zap/wrk/zap_report.html -f html
                    docker cp zap:/zap/wrk/zap_report.html .
                    docker stop zap
                    docker rm zap
                    """
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
                }
            }
        }
        stage('Send Report') {
            steps {
                script {
                    emailext(
                        to: params.PARAM_EMAIL,
                        subject: 'ZAP Security Scan Report',
                        body: 'Please find the attached ZAP security scan report.',
                        attachmentsPattern: 'zap_report.html'
                    )
                }
            }
        }
        stage('Parameters'){
            steps {
                echo "URL: ${params.PARAM_URL}"
                echo "Email: ${params.PARAM_EMAIL}"
            }
        }
    }
    post{
        always{
            cleanWs()
        } 
    }
}
