pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }  

      stage('unit test') {
            steps {
              sh "mvn test"
            }
            post {
              always {
                junit 'target/surefire-reports/*.xml'
                //jacoco execPattern: 'target/jacoco.exec'
                jacoco()

              }
            }
        } 
              stage('docker build and push') {
            steps {
              withDockerRegistry(credentialsId: 'dockerhub', url: '') {
              sh '''
              printenv
              docker build -t ch680351034/numeric-app:$GIT_COMMIT .
              docker push ch680351034/numeric-app:$GIT_COMMIT
              '''
              }

            }
         } 

    }
}