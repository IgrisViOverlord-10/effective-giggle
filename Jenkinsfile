pipeline {
    agent any

    tools {
        maven 'Maven'        // match your Jenkins tool name
        jdk 'Java21'         // match your installed JDK
    }

    environment {
        SONAR_PROJECT_KEY = 'petclinic'
        NEXUS_URL = 'http://13.204.63.137:8081'
        NEXUS_REPO = 'Maven_repository'
        TOMCAT_URL = 'http://3.108.40.229:8080'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/IgrisViOverlord-10/effective-giggle.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean install -Dcheckstyle.skip=true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                    -Dcheckstyle.skip=true
                    '''
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                script {
                    try {
                        sh 'trivy fs --exit-code 0 --no-progress .'
                    } catch (Exception e) {
                        echo "Trivy not installed or failed — skipping"
                    }
                }
            }
        }

stage('Upload to Nexus') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'nexus-creds',
            usernameVariable: 'NEXUS_USER',
            passwordVariable: 'NEXUS_PASS'
        )]) {
            sh '''
            curl -v -u admin:$NEXUS_PASS \
            --upload-file target/spring-petclinic-4.0.0-SNAPSHOT.jar \
            http://13.204.63.137:8081/repository/maven_repository_101/org/springframework/samples/spring-petclinic/4.0.0-SNAPSHOT/spring-petclinic-4.0.0-SNAPSHOT.jar
            '''
        }
    }
}

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat-creds', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                    curl -v -u $TOMCAT_USER:$TOMCAT_PASS \
                    -T target/*.jar \
                    "$TOMCAT_URL/manager/text/deploy?path=/petclinic&update=true"
                    '''
                }
            }
        }
    }
}
