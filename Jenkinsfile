pipeline {
   agent none 
    
    stages {
        stage('Compile et tests') {
            agent {
                kubernetes {
                    yamlFile 'kubernetesPod.yaml'
                }
            }
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
            post {
                always {
                    junit '**/surefire-reports/*.xml'
                }
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', followSymlinks: false
                    dir('application/target') {
                        stash includes: '*.jar', name: 'app'
                    }
                }
                unsuccessful {
                    mail bcc: '', body: 'Please correct ASAP', cc: '', from: '', replyTo: '', subject: 'Build failed', to: 'david.thibau@gmail.com'
                }
            }

             
        }
        stage('Analyse qualité et test intégration') {
            parallel {
                stage('Tests d integration') {
                     agent any
                     tools {
                        jdk 'JDK11'
                        maven 'MAVEN3'
                    }
                    steps {
                        sh 'mvn clean integration-test'
                    }
                    
                }
                 stage('Analyse Sonar') {
                      agent any
                      tools {
                        jdk 'JDK11'
                        maven 'MAVEN3'
                        }
                     steps {
                        sh 'mvn -Dsonar.login=admin -Dsonar.password=admin123 clean verify sonar:sonar'
                     }
                    
                }
            }
            
        }
            
        stage('Sélection DATACENTER') {
            agent none
            input {
                message 'Vers quel data center voulez vous déployer ?'
                ok 'Déployer'
                parameters {
                    choice choices: ['PARIS', 'NANCY', 'STRASBOURG'], name: 'DATACENTER'
                }
            }

            steps {
                echo "Déploiement intégration vers $DATACENTER"
                
            }
        }
        stage('Déploiement vers DockerHub') {
            agent any
          

            steps {
                unstash 'app'
                script {
                    def dockerImage = docker.build('dthibau/multi-module', '.')
                    docker.withRegistry('https://registry.hub.docker.com', 'dthibau_docker') {
                        dockerImage.push 'latest'
                    }
                }   
            }
        }

     }
    
}

