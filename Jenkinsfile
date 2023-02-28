pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes
  tools {
    maven 'my_maven'
  }

  environment {
    gitName = 'KSLeeDev'
    gitEmail = 'lllllks25@naver.com'
    gitWebaddress = 'https://github.com/KSLeeDev/jenk.git'
    gitSshaddress = 'git@github.com:KSLeeDev/jenk.git'
    gitCredential = 'git_cre' // github credential 생성 시의 ID
    dockerHubRegistry2 = 'leekyeongseo/supermario'
    dockerHubRegistry = 'leekyeongseo/springbootApp'
    dockerHubRegistryCredential = 'docker_cre'
  }

  stages {
    stage('checkout Github') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: gitCredential, url: gitWebaddress]]])
        }
        post {
            failure {
                echo 'Repository clone failure'
            }
            success {
                echo 'Repository clone success'
            }
        }
    }
    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
        // maven 플러그인이 미리 설치 되어있어야 함
        }
        post {
            failure {
                echo 'Maven build failure'
            }
            success {
                echo 'Maven build success'
            }
        }
    }
    stage('Docker image Build') {
      steps {
        sh "docker build -t ${dockerHubRegistry}:${currentBuild.number} ."
        sh "docker build -t ${dockerHubRegistry}:latest ."

        sh "docker build -t ${dockerHubRegistry2}:${currentBuild.number} -f Dockerfile2 ."
        sh "docker build -t ${dockerHubRegistry2}:latest -f Dockerfile2 ."
        }
        post {
            failure {
                echo 'docker image build failure'
            }
            success {
                echo 'docker image build success'
            }
        }
    }
    stage('Docker image Push') {
      steps {
        withDockerRegistry(credentialsId: dockerHubRegistryCredential, url: '') {
          // withDockerRegistry : docker pipeline 플러그인 설치시 사용가능.
          // dockerHubRegistryCredential : environment에서 선언한 docker_cre  
            sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry}:latest"

            sh "docker push ${dockerHubRegistry2}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry2}:latest"
          }
      }
      post {
        failure {
            echo 'Docker image push failure'
            sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker image rm -f ${dockerHubRegistry}:latest"

            sh "docker image rm -f ${dockerHubRegistry2}:${currentBuild.number}"
            sh "docker image rm -f ${dockerHubRegistry2}:latest"
        }
        success {
            echo 'Docker image push success'
            sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker image rm -f ${dockerHubRegistry}:latest"

            sh "docker image rm -f ${dockerHubRegistry2}:${currentBuild.number}"
            sh "docker image rm -f ${dockerHubRegistry2}:latest"
        }
     }
    }
    stage('k8s manifest file update') {
      steps {
        git credentialsId: gitCredential,
            url: gitWebaddress,
            branch: 'main'
        
        // 이미지 태그 변경 후 메인 브랜치에 푸시
        sh "git config --global user.email ${gitEmail}"
        sh "git config --global user.name ${gitName}"
        sh "sed -i 's@${dockerHubRegistry}:.*@${dockerHubRegistry}:${currentBuild.number}@g' deploy/sp-deploy.yml"
        sh "sed -i 's@${dockerHubRegistry2}:.*@${dockerHubRegistry2}:${currentBuild.number}@g' deploy/sup-deploy.yml"
        sh "git add ."
        sh "git commit -m 'fix:${dockerHubRegistry} ${currentBuild.number} image versioning'"
        sh "git commit -m 'fix:${dockerHubRegistry2} ${currentBuild.number} image versioning'"
        sh "git branch -M main"
        sh "git remote remove origin"
        sh "git remote add origin ${gitSshaddress}"
        sh "git push -u origin main"
      }
      post {
        failure {
          echo 'Container Deploy failure'
        }
        success {
          echo 'Container Deploy success'  
        }
      }
    }
  }
}
