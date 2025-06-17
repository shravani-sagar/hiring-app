pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/betawins/hiring-app.git'
        SONARQUBE_SERVER = 'SonarQube'         // Name as defined in Jenkins Global Tool Config
        MAVEN_HOME = tool 'Maven 3.8.7'         // Match tool name in Jenkins
        NEXUS_URL = 'http://your-nexus-url:8081'
        NEXUS_REPO = 'maven-releases'
        ARTIFACT_NAME = 'hiring-app'
        SLACK_CHANNEL = '#ci-cd'
        DEPLOY_PATH = '/var/lib/tomcat9/webapps/'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git url: "${env.GIT_REPO}", branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_SCANNER_HOME = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=${ARTIFACT_NAME} -Dsonar.sources=."
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean package"
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_URL}",
                    groupId: 'com.hiring',
                    version: '1.0.0',
                    repository: "${NEXUS_REPO}",
                    credentialsId: 'nexus-credentials-id', // Jenkins credentials
                    artifacts: [
                        [artifactId: "${ARTIFACT_NAME}", classifier: '', file: "target/${ARTIFACT_NAME}.war", type: 'war']
                    ]
                )
            }
        }

        stage('Notify Slack') {
            steps {
                slackSend channel: "${SLACK_CHANNEL}", message: "Build and Upload completed for ${ARTIFACT_NAME}"
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh """
                sudo cp target/${ARTIFACT_NAME}.war ${DEPLOY_PATH}
                sudo systemctl restart tomcat9
                """
            }
        }
    }

    post {
        success {
            slackSend channel: "${SLACK_CHANNEL}", message: "✅ Pipeline succeeded for ${ARTIFACT_NAME}"
        }
        failure {
            slackSend channel: "${SLACK_CHANNEL}", message: "❌ Pipeline failed for ${ARTIFACT_NAME}"
        }
    }
}
