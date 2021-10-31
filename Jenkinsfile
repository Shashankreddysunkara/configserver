pipeline {
  agent {
    kubernetes {
        defaultContainer 'gradle'
        yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            cicd: gradle-docker
        spec:
          containers:
          - name: gradle
            image: gradle:6.9.1-jdk8
            command:
            - cat
            tty: true
            volumeMounts:
            - name: build-cache
              mountPath: /tmp
          - name: docker
            image: docker:19.03.1-dind
            securityContext:
              privileged: true
          volumes:
          - name: build-cache
            hostPath:
            path: /tmp
        '''
    }
  }
  options {
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
    timeout(time: 60, unit: 'MINUTES')
    disableConcurrentBuilds()
  }
  environment {
    GITHUB = credentials('github-http-user-token')
  }
  stages {
    stage('Git Checkout') {
      steps {
        git url: 'https://github.com/ggnanasekaran77/configserver.git', branch: 'main'
      }      
    }

    stage('App Build') {
      steps {
        sh 'mkdir -p /tmp/cache'  
        sh 'gradle clean build -g /tmp/cache -x test'
      }
    }

    stage('Docker Build') {
      steps {
        container ('docker') {
          script {
            docker.withRegistry('', 'docker-hub-token') {
              def customImage = docker.build("ggnanasekaran77/configserver:${GIT_COMMIT}")
              customImage.push()
            }
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        sh "git clone https://$GITHUB_USR:$GITHUB_PSW@github.com/ggnanasekaran77/kube-demoapp.git"
        dir ("kube-demoapp") {
          sh """
            git config --global user.name 'Gnanasekaran Gajendiran'
            git config --global user.email 'ggnanasekaran77@gmail.com'
            sed -i "s#ggnanasekaran77/configserver:.*#ggnanasekaran77/configserver:${env.GIT_COMMIT}#" kube/deployment.yaml
            cat kube/deployment.yaml
            git commit -am 'Publish new version' && git push || echo 'no changes'
          """
        }
      }
    }

  }
}
