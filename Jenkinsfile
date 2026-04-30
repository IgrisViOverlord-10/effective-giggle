pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'Java21'
    }

    environment {
        SONAR_URL = "http://13.204.85.221:9000"
        NEXUS_URL = "http://13.204.63.137:8081"
        TOMCAT_URL = "http://3.108.40.229:8080"
    }

    stages {

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify -Dcheckstyle.skip=true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=petclinic \
                    -Dsonar.host.url=$SONAR_URL \
                    -Dsonar.login=$SONAR_AUTH_TOKEN \
                    -Dcheckstyle.skip=true
                    '''
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --exit-code 0 --severity HIGH,CRITICAL .'
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    curl -u $USER:$PASS --upload-file target/spring-petclinic-4.0.0-SNAPSHOT.jar \
                    $NEXUS_URL/repository/maven-releases/org/springframework/samples/petclinic/4.0.0/petclinic.jar
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    curl -u $USER:$PASS \
                    --upload-file target/spring-petclinic-4.0.0-SNAPSHOT.jar \
                    "$TOMCAT_URL/manager/text/deploy?path=/petclinic&update=true"
                    '''
                }
            }
        }
    }
}
