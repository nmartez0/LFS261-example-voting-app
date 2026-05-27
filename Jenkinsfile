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
            agent any

            tools {
              'NodeJS 22.4.0'
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
            agent any

            tools {
              'NodeJS 22.4.0'
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
        stage('Vote: build') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    args '--user root'
                }
            }
            
            when {
                changeset '**/vote/**'
            }

            steps {
                echo 'Compiling vote app.'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('Vote: test') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    args '--user root'
                }
            }

            when {
                changeset '**/vote/**'
            }

            steps {
                echo 'Running Unit Tests on vote app.'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('Vote: docker-package') {
            agent any

            when {
                branch 'master'
                changeset '**/vote/**'
              }

            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        // ./vote is the path to the Dockerfile that Jenkins will find from the Github repo
                        def voteImage = docker.build("xxxx/vote:v${env.BUILD_ID}", './vote')
                        voteImage.push()
                        //voteImage.push("${env.BRANCH_NAME}")
                        voteImage.push('latest')
                    }
                }
            }
        }

        post {
            always {
                echo 'Building multibranch pipeline for worker is completed.'
            }
        }
    }
}
