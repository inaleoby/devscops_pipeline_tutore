pipeline {
    agent any 

    stages {
        stage ('VM node version') {
            steps {
                sh '''
                    node -v 
                    npm -v
                
                '''
            }
        }
    }
}