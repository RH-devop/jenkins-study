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
                    sh 'npm run lint'
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