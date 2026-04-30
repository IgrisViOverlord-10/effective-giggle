pipeline {
    agent any

    tools {
    jdk 'Java21'
    maven 'Maven'
    }

    environment {
        SONAR_HOME = tool 'SonarQube'
        NEXUS_URL = 'http://localhost:8081'
        DOCKER_IMAGE = 'petclinic-app'
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/IgrisViOverlord-10/effective-giggle.git', branch: 'main'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package -Dcheckstyle.skip=true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=petclinic \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh '''
                    trivy fs . > trivy-report.txt
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh '''
                    echo "Uploading artifact to Nexus..."
                    mvn deploy -DskipTests
                '''
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                    cp target/*.war /opt/tomcat/webapps/petclinic.war
                '''
            }
        }
    }

    post {
        success {
            echo "PIPELINE SUCCESS"
        }
        failure {
            echo "PIPELINE FAILED"
        }
    }
}
