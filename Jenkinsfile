pipeline {
    agent any

    tools {
        nodejs 'nodejs'
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGO_DB_CRED = credentials('mongo-db-cred') //username & password separeted by : 
        MONGO_USERNAME = credentials('mongo-db-username')
        MONGO_PASSWORD = credentials('mongo-db-password')
        SONAR_SCANNER_HOME = tool 'sonarqube-scanner'
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

        stage ('OWASP Dependenccies Check') {

            steps {
                dependencyCheck additionalArguments: '''
                    --scan ./
                    --out ./
                    --format ALL
                    --disableYarnAudit \
                    --prettyPrint
                  ''', odcInstallation: 'owasp-dc'
                dependencyCheckPublisher failedTotalCritical: 2, pattern: 'dependency-check-reportxml', stopBuild: false // mettre a ture pour echouer le pipeline
    
                }
        }

        stage ('Test unitaire') {
            steps {
               

                sh 'npm test'
            }
        }


        stage ('Code Coverage') {
            steps {

                catchError(buildResult: 'SUCCESS', message: 'ERROR !! IT WILL BE FIXED IN NEXT VERSION', stageResult: 'UNSTABLE') {
                        sh 'npm run coverage'
                    }    
                 
        }
    
    }

    stage ('SAST - SonarQube') {
        steps {
               
                sh '''
                 $SONAR_SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=devsecops-tutore-project \
                    -Dsonar.sources=app.js \
                    -Dsonar.sonar.javascript.lcov.reportPaths=./coverage/lcov.info
                    -Dsonar.host.url=http://172.19.30.20:9090 \
                    -Dsonar.login=sqp_21233a29d9033487da21552754a24fa9712e4502
                
                '''
            }
        }



}

  post{

    always{

        junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml' // Rapport du dependency check sous forme de test, chaque vuln = test echoue

        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true]) //  Rapport du dependency check sous HTML

        junit allowEmptyResults: true, keepProperties: true, testResults: 'test-results.xml' // pour les test unitaire

        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report/', reportFiles: 'index.html', reportName: 'Code covergae HTML Report', reportTitles: '', useWrapperFileDirectly: true])  // Reprot for coverage
 
    }
  }


}