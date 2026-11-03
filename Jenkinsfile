pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {

        stage('Git checkout') {
            steps {
                git branch: 'main',
                    changelog: false,
                    poll: false,
                    url: 'https://github.com/prajvalg2110-max/java-student-crud-app.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.5.0.6356:sonar -Dsonar.projectKey=Student-crud'
                }
            }
        }

        stage('Build Maven application') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Docker build and push') {
            steps {
                script {
                    sh 'docker build -t java-student-app:v1 .'
                    sh 'docker tag java-student-app:v1 427064007413.dkr.ecr.ap-south-2.amazonaws.com/student-management-app:${BUILD_NUMBER}'
                    sh 'aws ecr get-login-password --region ap-south-2 | docker login --username AWS --password-stdin 427064007413.dkr.ecr.ap-south-2.amazonaws.com'
                    sh 'docker push 427064007413.dkr.ecr.ap-south-2.amazonaws.com/student-management-app:${BUILD_NUMBER}'
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh 'aws eks update-kubeconfig --name java-student-eks --region ap-south-2'
                sh 'kubectl set image deployment/java-student-app app=427064007413.dkr.ecr.ap-south-2.amazonaws.com/student-management-app:${BUILD_NUMBER}'
                sh 'kubectl rollout status deployment/java-student-app --timeout=120s'
            }
        }
    }
}