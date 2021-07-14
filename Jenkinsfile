pipeline {

  agent none


  stages{

  stage('worker build'){
    agent{
      docker{
        image 'maven:3.6.1-jdk-8-slim'
        args '-v $HOME/.m2:/root/.m2'
}
    }
    when {
      changeset "**/worker/**"
      branch 'master'
    }
    steps{
      echo 'building worker app'
      dir('worker'){
      sh 'mvn compile'
}
}
}

  stage('worker test'){
    agent{
      docker{
        image 'maven:3.6.1-jdk-8-slim'
        args '-v $HOME/.m2:/root/.m2'

      }
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'building worker app'
        dir(path: 'worker') {
          sh 'mvn compile'

        }

      }
    }


    stage('worker package') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'packaging worker app into a jarfile ,done'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('worker docker-package') {
      agent any
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'packaging worker image into a jarfile'
        script {
          docker.withRegistry('https://index.docker.io/v1/','dockerlogin'){
            def workerImage=docker.build("emnc/worker:v${env.BUILD_ID}","./worker")
            workerImage.push();
            workerImage.push("${env.BRANCH_NAME}");

          }
        }

      }
    }

    stage('vote build') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
        branch 'master'
      }
      steps {
        echo 'building vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('vote test') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
        branch 'master'
      }
      steps {
        echo 'running unit tests on vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }

      }
    }

    stage('vote docker-package') {
      agent any
      when {
        changeset '**/vote/**'
        branch 'master'
      }
      steps {
        echo 'packaging vote image into a jarfile'
        script {
          docker.withRegistry('https://index.docker.io/v1/','dockerlogin'){
            def voteImage=docker.build("emnc/vote:v${env.BUILD_ID}","./vote")
            voteImage.push();
            voteImage.push("${env.BRANCH_NAME}");

          }
        }

      }
    }

    stage('result build') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }

      }
      when {
        changeset '**/result/**'
        branch 'master'
      }
      steps {
        echo 'Compiling result app'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('result test') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }

      }
      when {
        changeset '**/result/**'
        branch 'master'
      }
      steps {
        echo 'Running'
        dir(path: 'result') {
          sh 'npm install'
          sh 'npm test'
        }

      }
    }

    stage('result docker-package') {
      agent any
      when {
        changeset '**/result/**'
        branch 'master'
      }
      steps {
        echo 'packaging result image into a jarfile'
        script {
          docker.withRegistry('https://index.docker.io/v1/','dockerlogin'){
            def resultImage=docker.build("emnc/result:v${env.BUILD_ID}","./result")
            resultImage.push();
            resultImage.push("${env.BRANCH_NAME}");

          }
        }

      }
    }

    stage('deploy to dev') {
      agent any
      steps {
        sh 'docker-compose up -d'
      }
    }


    stage('Sonarqube') {
      agent any

//    when{
//        branch 'master'
//      }

      tools {
        jdk "JDK11" // the name you have given the JDK installation in Global Tool Configuration
      }

      environment{
        sonarpath = tool 'SonarScanner'
      }

      steps {
            echo 'Running Sonarqube Analysis..'
            withSonarQubeEnv('sonar-instavote') {
              sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
            }
      }
    }


    stage("Quality Gate") {
        steps {
            timeout(time: 1, unit: 'HOURS') {
                // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                // true = set pipeline to UNSTABLE, false = don't
                waitForQualityGate abortPipeline: true
            }
        }
    }




}
  post{
    always{
      echo 'the mono pipeline job is complete'
}
}

  }
}
