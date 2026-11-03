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

                    sh 'docker build -t student-crud:v1 .'

                    sh 'docker tag student-crud:v1 427064007413.dkr.ecr.ap-south-2.amazonaws.com/student-management-app:${BUILD_NUMBER}'

                    sh 'aws ecr get-login-password --region ap-south-2 | docker login --username AWS --password-stdin 427064007413.dkr.ecr.ap-south-2.amazonaws.com'

                    sh 'docker push 427064007413.dkr.ecr.ap-south-2.amazonaws.com/student-management-app:${BUILD_NUMBER}'
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh '''
                    aws ecs describe-task-definition \
                        --task-definition java-ecs-task \
                        --query taskDefinition > task.json

                    jq --arg IMAGE "427064007413.dkr.ecr.ap-south-2.amazonaws.com/student-management-app:${BUILD_NUMBER}" \
                       --arg NAME "student-crud-container" \
                       '.containerDefinitions |= map(
                           if .name == $NAME then .image = $IMAGE else . end
                       )
                       | del(.taskDefinitionArn,.revision,.status,
                             .requiresAttributes,.compatibilities,
                             .registeredAt,.registeredBy)' \
                       task.json > new-task.json

                    TASK_ARN=$(aws ecs register-task-definition \
                        --cli-input-json file://new-task.json \
                        --query 'taskDefinition.taskDefinitionArn' \
                        --output text)

                    aws ecs update-service \
                        --cluster ekart-cluster \
                        --service java-ecs-task-service-9zmt9wog \
                        --task-definition "$TASK_ARN"

                    aws ecs wait services-stable \
                        --cluster ekart-cluster \
                        --services java-ecs-task-service-9zmt9wog
                '''
            }
        }
    }
}