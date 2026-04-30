pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK21'
    }

    environment {
        SONAR_HOME = "/opt/sonar-scanner"
        NEXUS_URL = "http://your-nexus:8081"
        DOCKER_IMAGE = "petclinic-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/IgrisViOverlord-10/effective-giggle.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
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

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh '''
                    trivy fs . --severity HIGH,CRITICAL
                '''
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh '''
                    curl -v -u admin:admin123 \
                    --upload-file target/*.war \
                    ${NEXUS_URL}/repository/maven-releases/petclinic.war
                '''
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                    cp target/*.war /opt/tomcat/webapps/petclinic.war
                    systemctl restart tomcat
                '''
            }
        }
    }

    post {
        success {
            echo "PIPELINE SUCCESS 🚀"
        }
        failure {
            echo "PIPELINE FAILED ❌"
        }
    }
}
