pipeline {
    agent any

    tools {
        nodejs 'nodejs'
    }

    stages {

        stage ('Repository Scanning') {
            steps {
                sh 'npm install --no-audit' 
            }
        }

        stage ('Install Dependencies') {
            steps {
                sh 'npm install --no-audit' 
            }
        }

        /*stage ('NPM Dependency Audit') {
            steps {
                sh '''
                    npm audit --audit-level=critical
                    echo $?
                    '''
            }
        }*/

        stage ('OWASP Dependenccies Check') {

            steps {
                dependencyCheck additionalArguments: '''
                    --scan ./
                    --out ./
                    --format XML
                    --disableYarnAudit \
                    --prettyPrint
                  ''', odcInstallation: 'owasp-dc'
                dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true               
                }
        }
        
    
    }
}