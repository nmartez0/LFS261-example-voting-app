pipeline {
    agent none

    stages {
        stage('Worker: build') {
            when {
                changeset '**/worker/**'
            }
            agent {
                docker {
                    image 'maven:3.9.16-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'compiling worker app...'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        stage('Worker: test') {
            when {
                changeset '**/worker/**'
            }
            agent {
                docker {
                    image 'maven:3.9.16-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Running Unit Tests on worker app'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        stage('Worker: package') {
            when {
                branch 'master'
                changeset '**/worker/**'
            }
            agent {
                docker {
                    image 'maven:3.9.16-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Packaging worker app into a jarfile'
                dir('worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('Worker: docker-package') {
            agent any
            when {
                branch 'master'
                changeset '**/worker/**'
            }
            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build('nmartez0/worker:v${env.BUILD_ID}', './worker')
                        workerImage.push()
                        // workerImage.push('build-${env.BUILD_ID}')
                        workerImage.push('latest')
                    }
                }
            }
        }

        stage('Result: build') {
            agent {
                docker {
                    image 'node:22.4.1-alpine'
                }
            }

            when {
                changeset '**/result/**'
            }
            steps {
                echo 'Compiling result app'
                dir('result') {
                    sh 'npm install'
                }
            }
        }
        stage('Result: test') {
            agent {
                docker {
                    image 'node:22.4.1-alpine'
                }
            }

            when {
                changeset '**/result/**'
            }
            steps {
                echo 'Runing Unit Test on result app'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
    }
    post {
        always {
            echo 'Building mono pipeline for vote app is completed.'
        }
    }
}
