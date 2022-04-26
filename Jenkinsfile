pipeline {
   agent none 
    
    stages {
        stage('Compile et tests') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-11-slim'
                    args '-v $HOME/.m2:/root/.m2'
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
        stage('Déploiement intégration') {
            agent any
          

            steps {
                unstash 'app'
                sh "mv *.jar ${env.DATACENTER}.jar"
                
            }
        }

     }
    
}

