pipeline {
    agent any

    tools {
        nodejs 'nodejs'
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGO_DB_CRED = credentials('mongo-db-cred') // username & password separated by :
        MONGO_USERNAME = credentials('mongo-db-username')
        MONGO_PASSWORD = credentials('mongo-db-password')
        SONAR_SCANNER_HOME = tool 'sonarqube-scanner'
    }

    stages {

        stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source ./ --exit-code 1'
            }
        }
        

        stage('Install Dependencies') {
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

        /*stage ('OWASP Dependencies Check') {
            steps {
                dependencyCheck additionalArguments: '''
                    --scan ./ 
                    --out ./ 
                    --format ALL 
                    --disableYarnAudit \
                    --prettyPrint
                ''', odcInstallation: 'owasp-dc'

                dependencyCheckPublisher failedTotalCritical: 2, pattern: 'dependency-check-report.xml', stopBuild: false // mettre à true pour échouer le pipeline
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
                timeout(time: 3600, unit: 'SECONDS') {
                    withSonarQubeEnv('sonarqube') {
                        sh '''
                            $SONAR_SCANNER_HOME/bin/sonar-scanner \
                                -Dsonar.projectKey=devsecops-tutore-project \
                                -Dsonar.sources=app.js \
                                -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info
                        '''
                    }
                    waitForQualityGate(abortPipeline: true)
                }
            }
        }*/

        stage('Build docker image') {
            steps {
                script {
                    try {
                        // Erreur volontaire pour tester le mail d’échec
                        sh 'docker build -t espoir10/devsecops-tutore:$GIT_COMMIT .' 
                    } catch (err) {
                        env.ERROR_STAGE = "DOCKER BUILD IMAGES"
                        env.ERROR_MESSAGE = err.getMessage()
                        error("Échec du build Docker : ${err.getMessage()}")
                    }
                }
            }
        }
        stage('Trivy vulnerabilty Scanner'){
            steps{
                sh '''
                    trivy image espoir10/devsecops-tutore:$GIT_COMMIT \
                        --severity LOW,MEDIUM \
                        --exit-code 0 \
                        --quiet \
                        --format json -o trivy-image-LOW-MEDIUM-results.json 

                    trivy image espoir10/devsecops-tutore:$GIT_COMMIT \
                        --severity HIGH,CRITICAL \
                        --exit-code 1 \
                        --quiet \
                        --format json -o trivy-image-CRITICAL-results.json 
            
                '''
            }

            post{
                always {
                    sh '''
                    trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                    --output trivy-image-LOW-MEDIUM-results.html trivy-image-LOW-MEDIUM-results.json 
                   
                   trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                    --output trivy-image-CRITICAL-results.html trivy-image-CRITICAL-results.json

                   trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                    --output trivy-image-LOW-MEDIUM-results.xml trivy-image-LOW-MEDIUM-results.json

                
                trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                    --output trivy-image-CRITICAL-results.xml trivy-image-CRITICAL-results.json


                    '''


                }
            }

        }

        stage('PUSH IMAGES'){
            steps{

                withDockerRegistry(credentialsId: 'DOCKER-HUB') {
                    sh 'docker push espoir10/devsecops-tutore:$GIT_COMMIT'

                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml' // Rapport du dependency check sous forme de test
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'HTML Report'])
            junit allowEmptyResults: true, keepProperties: true, testResults: 'test-results.xml' // tests unitaires
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'coverage/lcov-report/', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report'])

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'trivy-image-LOW-MEDIUM-results.html', reportName: 'Trivy image LM report'])

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'trivy-image-CRITICAL-results.html', reportName: 'Trivy image CRITICAL report'])
        }

        /*failure {
            script {
                emailext(
                    to: 'obympeespoir@gmail.com',
                    subject: "🚨 Échec du pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        <b>Pipeline échoué !</b><br>
                        🔹 <b>Stage :</b> ${env.ERROR_STAGE}<br>
                        🔹 <b>Erreur :</b> ${env.ERROR_MESSAGE}<br>
                        🔹 <b>Job :</b> ${env.JOB_NAME}<br>
                        🔹 <b>Build URL :</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a>
                    """,
                    mimeType: 'text/html'
                )
            }
        }*/
    }
}
