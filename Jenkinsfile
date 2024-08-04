pipeline {
    agent any
    
    stages {
        stage('SCA with OWASP Dependency Check') {
            steps {
                script {
                    def maxRetries = 3
                    def retryCount = 0
                    def success = false

                    while (!success && retryCount < maxRetries) {
                        try {
                            dependencyCheck additionalArguments: '''
                                --format HTML
                                --failOnCVSS 7
                                --prettyPrint
                            ''', odcInstallation: 'DP-Check'
                            success = true
                        } catch (Exception e) {
                            retryCount++
                            if (retryCount == maxRetries) {
                                echo "Warning: OWASP Dependency Check failed after ${maxRetries} attempts."
                                echo "Error: ${e.getMessage()}"
                                unstable "OWASP Dependency Check failed repeatedly. Please check NVD access and try again."
                            } else {
                                echo "OWASP Dependency Check failed. Retrying in 60 seconds... (Attempt ${retryCount} of ${maxRetries})"
                                sleep 60
                            }
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner';
                    withSonarQubeEnv('SonarQube scanner') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=newsread-microservice-application"
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker build -t karthick1616/newsread-customize-service customize-service/'
                    sh 'docker build -t karthick1616/newsread-news-service news-service/'
                }
            }
        }

        stage('Containerize And Test') {
            steps {
                script {
                    ['customize-service', 'news-service'].each { service ->
                        sh "docker run -d --name ${service} -e FLASK_APP=run.py karthick1616/newsread-${service} && sleep 10 && docker logs ${service} && docker stop ${service}"
                    }
                }
            }
        }

        stage('Push Images To Dockerhub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'DockerHubPass', variable: 'DOCKER_HUB_PASS')]) {
                        sh 'echo $DOCKER_HUB_PASS | docker login -u karthick1616 --password-stdin'
                        sh 'docker push karthick1616/newsread-news-service && docker push karthick1616/newsread-customize-service'
                    }
                }
            }
        }

        // Uncomment and adjust as needed
        // stage('Trivy scan on Docker images') {
        //     steps {
        //         sh 'TMPDIR=/home/jenkins'
        //         sh 'trivy image karthick1616/newsread-news:latest'
        //         sh 'trivy image karthick1616/newsread-customize:latest'
        //     }
        // }
    }    

    post {
        always {
            sh 'docker rm -f news-service || true'
            sh 'docker rm -f customize-service || true'
            // Publish the Dependency Check report
            publishHTML([allowMissing: false, 
                         alwaysLinkToLastBuild: true, 
                         keepAll: true, 
                         reportDir: '', 
                         reportFiles: 'dependency-check-report.html', 
                         reportName: 'Dependency Check Report', 
                         reportTitles: ''])
        }
        success {
            sh 'docker logout'   
        }
    }
}
