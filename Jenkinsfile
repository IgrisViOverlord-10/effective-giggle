pipeline {
    agent any

tools {
    maven 'Maven'
    jdk 'Java21'
    sonarScanner 'sonar-scanner'
}

    environment {
        SCANNER_HOME = tool 'sonar-scanner'

        SONAR_URL = 'http://13.204.85.221:9000'
        NEXUS_URL = 'http://13.204.63.137:8081'
        TOMCAT_URL = 'http://3.108.40.229:8080'

        IMAGE_NAME = 'spring-petclinic'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/IgrisViOverlord-10/effective-giggle.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=petclinic \
                        -Dsonar.host.url=$SONAR_URL \
                        -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs .'
            }
        }

        stage('Upload Artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-creds',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                    curl -v -u $NEXUS_USER:$NEXUS_PASS \
                    --upload-file target/spring-petclinic-4.0.0-SNAPSHOT.jar \
                    $NEXUS_URL/repository/maven-releases/spring-petclinic.jar
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'tomcat-creds',
                    usernameVariable: 'TOMCAT_USER',
                    passwordVariable: 'TOMCAT_PASS'
                )]) {
                    sh '''
                    curl -v -u $TOMCAT_USER:$TOMCAT_PASS \
                    --upload-file target/spring-petclinic-4.0.0-SNAPSHOT.jar \
                    "$TOMCAT_URL/manager/text/deploy?path=/petclinic&update=true"
                    '''
                }
            }
        }
    }
}
