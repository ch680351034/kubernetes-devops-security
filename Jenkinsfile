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

        stage('Sonar SAST') {
            steps {
              withSonarQubeEnv('sonarqube') {
              sh 'mvn sonar:sonar'
            }
          }
        } 
        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    //waitForQualityGate abortPipeline: true
                    script {
                    waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        stage('docker build and push') {
            steps {
              withDockerRegistry(credentialsId: 'dockerhub', url: '') {
              sh 'printenv'
              sh 'docker build -t ch680351034/numeric-app:$GIT_COMMIT .'
              sh 'docker push ch680351034/numeric-app:$GIT_COMMIT'
              sh 'docker image prune -fa'
              }

            }
         } 
            //$sed 's/unix/linux/' geekfile.txt
            stage('k8s deploy') {
            steps {
              withKubeConfig(credentialsId: 'k8s') {
              sh 'kubectl version --short'
              sh "sed -i 's#replace#ch680351034/numeric-app:$GIT_COMMIT#g' k8s_deployment_service.yaml"
              //sh "sed -i 's#replace#ch680351034/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yml"
              sh 'kubectl apply -f k8s_deployment_service.yaml'
              }

            }
         }

         

    }
}