pipeline {
    agent any

    stages {
        stage('Install') {
            steps {
                dir('app') {
                    sh 'npm ci'
                }
            }
        }

        stage('Lint') {
            steps {
                dir('app') {
                    sh 'exit 1'
                }
            }
        }

        stage('Build') {
            steps {
                dir('app') {
                    sh 'npm run build'
                }
            }
        }
    }
}