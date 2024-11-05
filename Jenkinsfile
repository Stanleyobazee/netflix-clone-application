pipeline{
    agent any
    tools{
        jdk 'jdk-17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Stanleyobazee/netflix-clone-application.gitt'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install && npm audit fix"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build --build-arg TMDB_V3_API_KEY=44bebaad0ac658671d1e448e13f2bb29 -t stanley80/netflix:${BUILD_NUMBER} ."
                       sh "docker push stanley80/netflix:${BUILD_NUMBER} "
                    }
                }
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker rm -f netflix || true' // Ensures the old container is removed if exists
                sh 'docker run -d --name netflix -p 8081:80 johntoby/netflix:${BUILD_NUMBER}'
            }
        } 
        stage("TRIVY"){
            steps{
                sh "trivy image johntoby/netflix:${BUILD_NUMBER} > trivyimage.txt"
            }
        }
    }

    post {
     always {
        emailext attachLog: false,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'borderlesspharma@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}



