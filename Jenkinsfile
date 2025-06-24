pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
       stage("Sonarqube Analysis") {
            steps {
        withSonarQubeEnv('sonar-server') {
            sh '''${SCANNER_HOME}/bin/sonar-scanner \
              -Dsonar.projectKey=Netflix \
              -Dsonar.projectName=Netflix \
              -Dsonar.projectVersion=1.0 \
              -Dsonar.sources=. \
              -Dsonar.host.url=http://13.201.131.240:9000 \
              -Dsonar.login=sqp_0c10d4a977f3a715567f46ce97a5dc268131226e'''
        }
    }
}
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
  steps {
    sh 'npm cache clean --force'  
    sh 'npm install'
  }
}
    }
}
    