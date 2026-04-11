pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIAL')
        VERSION = "${env.BUILD_ID}"
    }

    tools {
        maven "Maven"
    }

    stages {

        // ==================== COMMON STAGES (Run on ALL branches) ====================
        stage('Checkout Code') {
            steps {
                echo "Checking out code from branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Maven Clean Install') {
            steps {
                echo "Running Maven Clean Install on branch: ${env.BRANCH_NAME}"
                sh 'mvn clean install'
            }
        }

        // ==================== DEPLOYMENT STAGES (Only on Master) ====================
        stage('Docker Build and Push') {
            when {
                branch 'master'
            }
            steps {
                echo "Starting Docker Build and Push stage..."

                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'

                sh "docker build -t dockersiva777/restaurant-listing-service:${VERSION} ."
                sh "docker push dockersiva777/restaurant-listing-service:${VERSION}"

                echo "Docker image pushed successfully with tag: ${VERSION}"
            }
        }

        stage('Update Image Tag in GitOps') {
            when {
                branch 'master'
            }
            steps {
                echo "Starting GitOps Update stage..."

                checkout scmGit(branches: [[name: '*/master']],
                                extensions: [],
                                userRemoteConfigs: [[ credentialsId: 'ssh-privatekey',
                                                      url: 'git@github.com:ReactJava1918/deployment-folder-master.git']])

                sh """
                    sed -i "s|image:.*|image: dockersiva777/restaurant-listing-service:${VERSION}|" aws/restaurant-manifest.yml
                """

                sh 'git checkout master'
                sh 'git add .'
                sh 'git commit -m "Update image tag to ${VERSION} by Jenkins" || echo "No changes to commit"'

                sshagent(['ssh-privatekey']) {
                    sh 'git push origin master'
                }

                echo "GitOps repository updated successfully!"
            }
        }

    }

    post {
        always {
            echo "Pipeline completed for branch: ${env.BRANCH_NAME}"
        }
        success {
            echo "✅ Pipeline succeeded for ${env.BRANCH_NAME}"
        }
        failure {
            echo "❌ Pipeline failed for ${env.BRANCH_NAME}"
        }
    }
}