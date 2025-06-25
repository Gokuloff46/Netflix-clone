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
                git branch: 'main', url: 'https://github.com/Gokuloff46/Netflix-clone.git'
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
              -Dsonar.host.url=http://3.110.221.90:9000 \
              -Dsonar.login=sqp_519ec7c4c2bf3ab0d4ad8266e2dddd0085268579'''
        }
    }
}
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
  steps {
    sh 'npm cache clean --force' 
    sh 'npm install'
  }
}

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=4da015bba809123ef4a72acc034c5bed -t netflix ."
                       sh "docker tag netflix gokulfgi/netflix:latest "
                       sh "docker push gokulfgi/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image gokulfgi/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d \
                    --restart unless-stopped \
                    -p 80:80 \
                     gokulfgi/netflix:latest'
            }
        }
    
}
}

