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

    stage('Maven Build') {
        steps {
            echo "Starting Maven Build stage..."
            sh 'mvn clean package -DskipTests'
            echo "Maven Build completed successfully."
        }
    }

    stage('Run Tests') {
      steps {
          echo "Starting Run Tests stage..."
          sh 'mvn test'
          echo "Tests executed successfully."
      }
    }

    /*
    stage('SonarQube Analysis') {
      steps {
        echo "Starting SonarQube Analysis..."
        sh 'mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install sonar:sonar -Dsonar.host.url=http://35.180.137.8:9000/ -Dsonar.login=squ_32789bcdadb6e4337e432d6cbc100c2a1a14fde5'
        echo "SonarQube Analysis completed."
      }
    }

    stage('Check code coverage') {
      steps {
        script {
            echo "Starting Code Coverage Check..."

            def token = "squ_32789bcdadb6e4337e432d6cbc100c2a1a14fde5"
            def sonarQubeUrl = "http://35.180.137.8:9000/api"
            def componentKey = "com.codeddecode:restaurantlisting"
            def coverageThreshold = 80.0

            def response = sh (
                script: "curl -H 'Authorization: Bearer ${token}' '${sonarQubeUrl}/measures/component?component=${componentKey}&metricKeys=coverage'",
                returnStdout: true
            ).trim()

            def coverage = sh (
                script: "echo '${response}' | jq -r '.component.measures[0].value'",
                returnStdout: true
            ).trim().toDouble()

            echo "Coverage: ${coverage}%"

            if (coverage < coverageThreshold) {
                error "Coverage is below the threshold of ${coverageThreshold}%. Aborting the pipeline."
            } else {
                echo "Code coverage check passed. Coverage is ${coverage}%"
            }
        }
      }
    }
    */

    stage('Docker Build and Push') {
      steps {
          echo "Starting Docker Build and Push stage..."

          echo "Logging into Docker Hub..."
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'

          echo "Building Docker image with tag ${VERSION}..."
          sh 'docker build -t dockersiva777/restaurant-listing-service:${VERSION} .'

          echo "Pushing Docker image to Docker Hub..."
          sh 'docker push dockersiva777/restaurant-listing-service:${VERSION}'

          echo "Docker Build and Push completed successfully."
      }
    }

    stage('Cleanup Workspace') {
      steps {
          echo "Starting Workspace Cleanup..."
          deleteDir()
          echo "Workspace cleanup completed."
      }
    }

    stage('Update Image Tag in GitOps') {
      steps {
          echo "Starting GitOps Update stage..."

          echo "Checking out GitOps repository..."
          checkout scmGit(branches: [[name: '*/master']],
                          extensions: [],
                          userRemoteConfigs: [[ credentialsId: 'git-ssh',
                                                url: 'git@github.com:ReactJava1918/deployment-folder-master.git']])

          echo "Updating image tag in aws/restaurant-manifest.yml to version ${VERSION}..."
          sh '''
            sed -i "s/image:.*/image: dockersiva777\\/restaurant-listing-service:${VERSION}/" aws/restaurant-manifest.yml
          '''

          echo "Committing and pushing changes to Git..."
          sh 'git checkout master'
          sh 'git add .'
          sh 'git commit -m "Update image tag to ${VERSION}" || echo "No changes to commit"'

          sshagent(['git-ssh']) {
              sh 'git push'
          }

          echo "GitOps repository updated successfully with new image tag."
      }
    }

  }
}