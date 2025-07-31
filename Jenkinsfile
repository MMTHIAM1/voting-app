pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarLocal'                 // Nom configuré dans Jenkins > SonarQube Servers
        DOCKERHUB_CREDS = 'docker-hub-creds'            // ID des credentials dans Jenkins
        DOCKERHUB_REPO = 'mon_repository'               // À adapter à ton Docker Hub
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Clonage du dépôt Git...'
                git url: 'https://github.com/MMTHIAM1/voting-app.git', branch: 'main'
                echo '✅ Code source récupéré avec succès.'
            }
        }

        stage('Build Docker Images') {
            steps {
                dir('voting-app') {
                    script {
                        echo '🔧 Build des images Docker...'
                        sh 'docker-compose build'
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                dir('voting-app') {
                    script {
                        echo '🚀 Déploiement de l’application avec Docker Compose...'
                        sh 'docker-compose up -d'
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                dir('voting-app') {
                    script {
                        echo '🔍 Vérification du déploiement...'
                        sh 'docker-compose ps'
                        sleep(time: 10, unit: 'SECONDS')
                        echo '✅ Tous les services semblent déployés.'
                    }
                }
            }
        }

        

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDS}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    dir('voting-app') {
                        echo '📦 Push des images Docker sur Docker Hub...'
                        sh '''
                            echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin

                            docker tag job5-vote $DOCKERHUB_USER/${DOCKERHUB_REPO}:vote
                            docker tag job5-result $DOCKERHUB_USER/${DOCKERHUB_REPO}:result
                            docker tag job5-worker $DOCKERHUB_USER/${DOCKERHUB_REPO}:worker

                            docker push $DOCKERHUB_USER/${DOCKERHUB_REPO}:vote
                            docker push $DOCKERHUB_USER/${DOCKERHUB_REPO}:result
                            docker push $DOCKERHUB_USER/${DOCKERHUB_REPO}:worker
                        '''
                    }
                }
            }
        }
        stage('SonarQube analysis') {
            steps {
                // Utilise le wrapper pour les credentials et les variables d'environnement SonarQube
                withSonarQubeEnv('Mon Serveur SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=mon_projet_devops -Dsonar.sources=."
                }
            }
        }

        // Étape optionnelle pour attendre que l'analyse soit complète
        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    // La fonction waitForQualityGate va attendre que l'analyse SonarQube soit traitée
                    // et que le "Quality Gate" soit vert avant de continuer.
                    // Elle échouera après 5 minutes si ce n'est pas le cas.
                    sh 'sleep 10' // Attendre un peu avant de faire la première requête
                    // Utilise le nom de votre serveur SonarQube que vous avez configuré
                    script {
                        def scannerHome = tool 'SonarQubeScanner';
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=mon_projet_devops -Dsonar.sources=."
                        waitForQualityGate()
                    }
                }
            }
        }
    }
    }

    post {
        success {
            mail to: 'mactaribrahimathiam@gmail.com',
                 subject: "✅ SUCCESS - Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Le pipeline a réussi !\n\nVoir les détails ici : ${env.BUILD_URL}"
        }
        failure {
            mail to: 'mactaribrahimathiam@gmail.com',
                 subject: "❌ FAILURE - Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Le pipeline a échoué.\n\nConsulte la console ici : ${env.BUILD_URL}"
        }
        always {
            echo '🧹 Nettoyage du workspace...'
            deleteDir()
        }
    }
}

