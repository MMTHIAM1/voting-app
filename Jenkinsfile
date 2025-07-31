pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/MMTHIAM1/voting-app.git', branch: 'main'
                echo 'Source code checked out successfully.'
            }
        }

        stage('Build Docker Images') {
            steps {
                dir('voting-app') {
                    script {
                        echo 'Building Docker images for all services...'
                        sh 'docker-compose build'
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                dir('voting-app') {
                    script {
                        echo 'Deploying the application stack...'
                        sh 'docker-compose up -d'
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                dir('voting-app') {
                    script {
                        echo 'Verifying the status of deployed services...'
                        sh 'docker-compose ps'
                        sleep(time: 10, unit: 'SECONDS')
                        echo 'Deployment verification complete.'
                    }
                }
            }
        }
    }

    post {
    	success {
    		mail to: 'your@email.com',
         	subject: "Pipeline SUCCESS",
         	body: "Le pipeline a réussi !"
  	}
  	failure {
    		mail to: 'your@email.com',
         	subject: "Pipeline FAILURE",
         	body: "Le pipeline a échoué."
        }
        always {
            echo 'Pipeline finished. Cleaning up workspace.'
        }
    }
}
