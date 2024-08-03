pipeline {
    agent any
    
    stages{
        stage('SCA with OWASP Dependency Check') {
        steps {
            dependencyCheck additionalArguments: '''--format HTML
            ''', odcInstallation: 'DP-Check'
            }
    }

        stage('SonarQube Analysis') {
      steps {
        script {
          // requires SonarQube Scanner 2.8+
          scannerHome = tool 'SonarScanner'
        }
        withSonarQubeEnv('Sonarqube Server') {
          sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=newsread-microservice-application"
        }
      }
    }

       stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker build -t karthick1616/newsread-customize customize-service/'
                    sh 'docker build -t karthick1616/newsread-news news-service/'
                }
            }
        }

        stage('Containerize And Test') {
            steps {
                script {
                    sh 'docker run -d --name customize-service -e FLASK_APP=run.py k1616/newsread-customize || echo "Failed to run customize-service"'
                    sh 'docker ps -a'
                    sh 'docker logs customize-service || echo "No logs for customize-service"'
                    sh 'docker stop customize-service || echo "Failed to stop customize-service"'

                    sh 'docker run -d --name news-service -e FLASK_APP=run.py k1616/newsread-news || echo "Failed to run news-service"'
                    sh 'docker ps -a'
                    sh 'docker logs news-service || echo "No logs for news-service"'
                    sh 'docker stop news-service || echo "Failed to stop news-service"'
                }
            }
        }

        stage('Push Images To Dockerhub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'DockerHubPass', variable: 'DockerHubPass')]) {
                        sh 'docker login -u karthick1616 --password $DockerHubPass'
                    }
                    sh 'docker push karthick616/newsread-news'
                    sh 'docker push karthick1616/newsread-customize'
                }
            }
        }
        //stage('Trivy scan on Docker images'){
          //  steps{
            //     sh 'TMPDIR=/home/jenkins'
              //   sh 'trivy image kelvinskell/newsread-news:latest'
                // sh 'trivy image kelvinskell/newsread-customize:latest'
        //}
       
   // }
        }    

      post {
        always {
            script {
                def containers = ['news-service', 'customize-service']
                containers.each { container ->
                    sh "docker ps -a | grep ${container} && docker rm ${container} || echo 'Container ${container} does not exist'"
                }
            }
        }
        success {
            sh 'docker logout'
        }
    }
}
