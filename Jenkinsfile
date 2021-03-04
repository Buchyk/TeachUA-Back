pipeline {
    agent {
        docker {
            image 'maven:3-openjdk-8' 
            args '-v /root/.m2:/root/.m2' 
        }
    }
    stages {
        stage('Build') { 
            steps {
                echo 'Running build automation'
                sh 'mvn -B -DskipTests clean package' 
                archiveArtifacts artifacts: 'dist/dev.zip'
            }
        }
    }
}
