pipeline {
    agent {
        label 'maven'
    }

    options {
        buildDiscarder logRotator(numToKeepStr: '10')
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
    }

    parameters {
        booleanParam name: 'CLEAN_BEFORE', description: 'Executa o passo de "clean" antes do build'
    }

    environment {
        MAVEN_OPTS='-XX:TieredStopAtLevel=1 -XX:+UseParallelGC'
    }

    stages {
        stage('Clean') {
            when {
                anyOf {
                    expression { params.CLEAN_BEFORE }
                    branch 'master'
                }
            }
            steps {
               mvn 'clean'
            }
        }

        stage('Build') {
            steps {
                mvn 'compile'
            }
        }

        stage('Test') {
            steps {
                mvn 'verify'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                script {
                    artifactId = readPom('project.artifactId')
                    version = readPom('project.version')

                    withCredentials([string(credentialsId: 'super-deploy-secret', variable: 'SUPER_CREDENTIALS')]) {
                        sh "./super-deploy.sh $artifactId $version"
                    }
                }
            }
            post {
                success {
                    script {
                        currentBuild.description = "Deployed $artifactId version $version"
                    }
                }
            }
        }
    }
}

def mvn(String args) {
    sh "mvn --no-transfer-progress -B $args"
}

def readPom(String expression) {
    return sh(script: """mvn help:evaluate -Dexpression="$expression" -q -DforceStdout""", returnStdout: true)
}