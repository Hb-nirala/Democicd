pipeline {
    agent any

    environment {
        ANDROID_HOME = "/Users/hb/Library/Android/sdk"
        JAVA_HOME    = "/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home"
        NODE_BIN = "/Users/hb/.nvm/versions/node/v22.20.0/bin"
        PATH = "${NODE_BIN}:${env.PATH}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Verify Node') {
            steps {
            sh '''
            which node
            which npm
            node -v
            npm -v
            '''
            }
        }

        stage('Install Node Dependencies') {
            steps {
                sh '''
                npm install
                '''
            }
        }

        stage('Clean Android Build') {
            steps {
                sh '''
                cd android
                ./gradlew clean
                '''
            }
        }

        stage('Build Android Debug APK') {
            steps {
                sh '''
                cd android
                ./gradlew assembleDebug
                '''
            }
        }

        stage('Collect APK Artifact') {
            steps {
                sh '''
                mkdir -p build-output
                cp android/app/build/outputs/apk/debug/app-debug.apk build-output/
                '''
            }
        }

        stage('Archive APK') {
            steps {
                archiveArtifacts artifacts: 'build-output/*.apk', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "✅ APK build ready — download from Jenkins artifacts and install on your device"
        }

        failure {
            echo "❌ Build failed — check Jenkins console logs"
        }
    }
}
