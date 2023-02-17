def dataCenter = ''

def log(msg) {
    println msg
}

pipeline {

   agent none
   tools {
    jdk 'JDK14'
    maven 'MAVEN3'
   } 


    stages {
        stage('Compile et tests') {
            agent any 
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
                    script {
                        log('Coucou')
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
                    steps {
                        echo 'Tests d integration'
                        sh "mvn clean integration-test"
                    }
                    
                }
                stage('Analyse Sonar') {
                    agent any
                    environment {
                        SONAR_TOKEN = credentials('JETON_SONAR')
                    }
                     steps {
                        echo 'Analyse sonar'
                        sh "mvn -Dsonar.login=$SONAR_TOKEN clean verify sonar:sonar"
                     }    
                }
                stage('Dependecy scanning') {
                    agent any
                    steps {
                        sh 'mvn -DskipTests=true clean package dependency-check:check'
                    }
                    post {
                         always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target', reportFiles: 'dependency-check-report.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                         }
                    }
                }
            }
            
        }
            
        stage('Sélection DATACENTER') {
            agent none
            options {
                timeout(5)
            }
            input {
                message 'Vers quel data center voulez vous déployer ?'
                ok 'Déployer'
                parameters {
                    choice choices: ['PARIS', 'NANCY', 'STRASBOURG'], name: 'DATACENTER'
                }
            }

            steps {
                echo "Déploiement intégration vers $DATACENTER"
                script {
                    dataCenter = "$DATACENTER"
                }

                
            }
        }
        stage('Déploiement intégration') {
            agent any
          
            steps {
                cleanWs()
                unstash 'app'
                sh 'ls'
                sh "mv *application*.jar ${dataCenter}.jar"
                
            }
        }


     }
    
}

