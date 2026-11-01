def commit_id
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh "git rev-parse --short HEAD > .git/commit-id"
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }
        stage('Build') {
            steps {
                echo 'Building....'
                sh 'JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64 ./mvnw -B -DskipTests clean package'
                echo 'build complete'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('code-quality') {
                    sh 'JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64 ./mvnw -B org.sonarsource.scanner.maven:sonar-maven-plugin:5.7.0.6970:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_AUTH_TOKEN'
                }
            }
        }
        stage('Build image') {
            steps {
                echo 'Building docker image....'
                sh "docker build -t position-simulator:${commit_id} ."
            }
        }
        stage('Push image') {
            steps {
                echo 'Pushing image to the cluster....'
                sh "minikube image load position-simulator:${commit_id}"
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes'
                sh "sed -i -r 's|richardchesterwood/k8s-fleetman-position-simulator:release2|position-simulator:${commit_id}|' ./manifests/workloads.yaml"
                sh 'kubectl apply -f ./manifests/'
                sh 'kubectl get all'
                echo 'deployment complete'
            }
        }
    }
    post {
        success {
            echo "Pipeline termine avec succes : position-simulator:${commit_id}"
        }
        failure {
            echo 'Echec du pipeline : consulter la console Jenkins'
        }
    }
}
