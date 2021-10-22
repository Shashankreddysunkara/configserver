pipeline {
    agent any
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
        stage('Build') {
            steps {
                sh "./gradlew clean build"
            }
        }

        stage('Docker') {
            steps {
                script {
                    def customImage = docker.build("ggnanasekaran77/configserver:${env.GIT_COMMIT}")
                    customImage.push()

                }
            }
        }

        stage('Deploy') {
            steps {
                sh "git clone https://$GITHUB_USR:$GITHUB_PSW@github.com/ggnanasekaran77/kube-demoapp.git"
                dir ("kube-demoapp") {
                    sh "git config --global user.name 'Gnanasekaran Gajendiran'"
                    sh "git config --global user.email 'ggnanasekaran77@gmail.com'"
                    sh "sed -i 's/replicas: 1/replicas: 2/g' kube/deployment.yaml"
                    sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
                }
            }
        }
    }
    post {
        always {
            cleanWs()
            dir("${env.WORKSPACE}@tmp") {
                deleteDir()
            }
            dir("${env.WORKSPACE}@script") {
                deleteDir()
            }
            dir("${env.WORKSPACE}@script@tmp") {
                deleteDir()
            }
        }
    }
}