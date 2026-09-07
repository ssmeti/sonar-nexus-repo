pipeline {
    agent any

    environment {
            PATH = "/usr/local/Cellar/maven/3.9.16/libexec/bin:/usr/local/Cellar/openjdk/26.0.2.1/libexec/openjdk.jdk/Contents/Home/bin:${env.PATH}"
        SONAR_HOST_URL = 'http://localhost:9000'
        NEXUS_URL      = 'localhost:18081'
        NEXUS_REPO     = 'maven-releases'
        NEXUS_CRED_ID  = 'nexus-credentials'
        SONAR_TOKEN    = credentials('sonar-token')  // stored in Jenkins credentials
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ssmeti/sonar-nexus-repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarServer') {  // name from Jenkins SonarQube config
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=my-app \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_URL}",
                    groupId: 'com.example',
                    version: '1.0.${BUILD_NUMBER}',
                    repository: "${NEXUS_REPO}",
                    credentialsId: "${NEXUS_CRED_ID}",
                    artifacts: [
                        [artifactId: 'my-app',
                         classifier: '',
                         file: 'target/my-app-1.0-SNAPSHOT.jar',
                         type: 'jar']
                    ]
                )
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully — code analyzed and artifact pushed to Nexus.'
        }
        failure {
            echo 'Pipeline failed — check console log.'
        }
    }
}
