pipeline {
    options { timestamps() }
    agent {
        docker {
            image 'maven:3-openjdk-8'
            args '-v /root/.m2:/root/.m2' 
        }
    }
   
    stages {
        stage('Test') { 
            steps {
                echo 'Running automation test'
                sh 'mvn test'
            }
        }
        stage('Build') { 
            steps {
                echo 'Running build automation'
                sh 'mvn clean package'
                sh 'tar czf app-${BUILD_NUMBER}.tar.gz target/*.war' 
            }
        }
         stage('Push to artifact server') { 
             steps {
                withCredentials([usernamePassword(credentialsId: 'artifact', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'artifact_server',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'app-${BUILD_NUMBER}.tar.gz',
                                        remoteDirectory: '/artifactory',
                                        
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    stage('DeployToStaging') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'backend', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'backend_root',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'target/TeachUA-1.0.war',
                                        removePrefix: 'target/',
                                        remoteDirectory: '/home/teachua/www/back',
                                         execCommand: 'sudo mv /home/teachua/www/back/TeachUA-1.0.war /home/teachua/www/back/dev.war'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}
