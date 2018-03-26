pipeline {
  agent {
    docker {
      image 'maven:3-alpine'
      args '-v /root/.m2:/root/.m2'
    }
    
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn -B -DskipTests clean package'
      }
      post {
        always {
          archiveArtifacts(artifacts: '**/*.jar', fingerprint: true)
          
        }
        
      }
    }
    stage('Test') {
      parallel {
        stage('Test') {
          steps {
            sh 'mvn test'
          }
          post {
            always {
              junit 'target/surefire-reports/*.xml'
              
            }
            
          }
        }
        stage ("SonarQube analysis") {
          steps {
            withSonarQubeEnv('sonarqube') {
              sh "../../../sonar-scanner-2.9.0.670/bin/sonar-scanner"
            }
            def qualitygate = waitForQualityGate()
            if (qualitygate.status != "OK") {
              error "Pipeline aborted due to quality gate coverage failure: ${qualitygate.status}"
            }
          }
        }
      }
    }
    stage('Deliver') {
      steps {
        sh './jenkins/scripts/deliver.sh'
      }
    }
  }
}
