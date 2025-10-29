pipeline {
    agent any

    tools {
        nodejs 'nodejs'
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
    }

    stages {

        stage ('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source ./ --exit-code 1' 
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

        /*stage ('OWASP Dependenccies Check') {

            steps {
                dependencyCheck additionalArguments: '''
                    --scan ./
                    --out ./
                    --format ALL
                    --disableYarnAudit \
                    --prettyPrint
                  ''', odcInstallation: 'owasp-dc'
                dependencyCheckPublisher failedTotalCritical: 2, pattern: 'dependency-check-reportxml', stopBuild: false // mettre a ture pour echouer le pipeline

                junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml' // Rapport du dependency check sous forme de test, chaque vuln = test echoue

                publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])              
                }
        }

        stage ('Test unitaire') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mongo-db-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {

                         sh 'npm test' 
                    }
                junit allowEmptyResults: true, stdioRetention: '', testResults: 'test-results.xml' // pour les test unitaire
            }
        }*/


        stage ('Code Coverage') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mongo-db-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {

                    catchError(buildResult: 'SUCCESS', message: 'ERROR !! IT WILL BE FIXED IN NEXT VERSION', stageResult: 'UNSTABLE') {
                        sh 'npm run coverage'
                    }    
                }
                
        }
        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report/', reportFiles: 'index.html', reportName: 'Code covergae HTML Report', reportTitles: '', useWrapperFileDirectly: true])  // Reprot for coverage


        
    
    }
}

}