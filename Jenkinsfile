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
                container('maven') {
                    sh 'mvn -Dmaven.test.failure.ignore=true clean package'
                }
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
    }
 
    
}

