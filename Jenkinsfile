pipeline {
  agent any
  options { timestamps() }

  environment {
    IMAGE = 'ayajguirim/monapp'
    TAG   = "build-${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Docker Build') {
      steps {
        bat 'docker version'
        bat "docker build -t %IMAGE%:%TAG% ."
      }
    }

    stage('Smoke Test') {
      steps {
        bat """
          docker rm -f monapp_test 2>nul || ver > nul
          docker run -d --name monapp_test -p 8085:80 %IMAGE%:%TAG% || exit /b 1
          ping -n 4 127.0.0.1 > nul
          curl -sI http://localhost:8085 | find "200 OK" || (docker rm -f monapp_test & exit /b 1)
          docker rm -f monapp_test
        """
      }
    }

    stage('Push (Docker Hub)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                          usernameVariable: 'USER',
                                          passwordVariable: 'PASS')]) {
          bat """
            echo %PASS%| docker login -u %USER% --password-stdin || exit /b 1
            docker tag %IMAGE%:%TAG% %IMAGE%:latest
            docker push %IMAGE%:%TAG% || exit /b 1
            docker push %IMAGE%:latest || exit /b 1
          """
        }
      }
    }

    stage('Deploy') {
      steps {
        bat """
          docker compose pull || exit /b 1
          docker compose up -d || exit /b 1
          docker ps
        """
      }
    }
  }

  post {
    success { echo 'Build+Test+Push+Deploy OK' }
    failure { echo 'Build/Tests/Push/Deploy KO' }
  }
}