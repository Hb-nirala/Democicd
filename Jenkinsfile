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
    }

    post {
        success {
            emailext(
                subject: "✅ APK Build Success - ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                    <h2>✅ React Native APK Build Successful</h2>

                    <p><b>Job:</b> ${JOB_NAME}</p>
                    <p><b>Build:</b> #${BUILD_NUMBER}</p>
                    <p><b>Status:</b> ${currentBuild.currentResult}</p>

                    <p>You can view the build details here:</p>
                    <a href="${BUILD_URL}">${BUILD_URL}</a>
                """,
                to: "nirala.kumar@hiddenbrains.in",
                mimeType: "text/html"
            )
        }
        failure {
            emailext(
                subject: "❌ APK Build FAILED - ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                    <h2>❌ React Native APK Build FAILED</h2>

                    <p><b>Job:</b> ${JOB_NAME}</p>
                    <p><b>Build:</b> #${BUILD_NUMBER}</p>
                    <p><b>Status:</b> ${currentBuild.currentResult}</p>

                    <p>Check the console logs:</p>
                    <a href="${BUILD_URL}console">${BUILD_URL}console</a>
                """,
                to: "nirala.kumar@hiddenbrains.in",
                mimeType: "text/html"
            )
        }
    }
}
