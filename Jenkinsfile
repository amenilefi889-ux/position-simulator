def commit_id
pipeline {
    agent any

    stages {
        stage('Preparation') {
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
        stage('Code Quality') {
            steps {
                withSonarQubeEnv('code-quality'){
                    sh 'JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64 ./mvnw -B org.sonarsource.scanner.maven:sonar-maven-plugin:5.7.0.6970:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_AUTH_TOKEN'
                }
            }
        }
        stage('Image Build') {
            steps {
                echo 'Building docker image....'
                sh 'scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -r -i $(minikube ssh-key) ./*  docker@$(minikube ip):~/'
                sh "minikube image build -t position-simulator:${commit_id} ."
                echo 'image build complete'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying to Kubernetes'
                sh "sed -i -r 's|richardchesterwood/k8s-fleetman-position-simulator:release2|position-simulator:${commit_id}|' ./manifests/workloads.yaml"
                sh 'kubectl apply -f ./manifests/'
                sh 'kubectl get all'
                echo 'deployment complete'
            }
        }
    }
}
