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

        stage('Send Email with APK') {
            steps {
                emailext(
                subject: "✅ APK Build Ready - ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                    <h2>✅ React Native Debug APK Ready</h2>

                    <p><b>Job:</b> ${JOB_NAME}</p>
                    <p><b>Build:</b> #${BUILD_NUMBER}</p>
                    <p><b>Status:</b> ${currentBuild.currentResult}</p>

                    <p>Download Jenkins console:</p>
                    ${BUILD_URL}

                    <br><br>
                    <b>APK attached.</b>
                    """,
                    to: "nirala.kumar@hiddenbrains.in",
                    attachmentsPattern: "build-output/*.apk",
                    mimeType: "text/html"
                )
            }
        }
    }


    post {
        success {
            emailext(
            subject: "✅ APK Build Ready",
            body: "APK attached — Jenkins build ${BUILD_NUMBER}",
            to: "nirala.kumar@hiddenbrains.in",
            attachmentsPattern: "build-output/*.apk"
        )
        }

        failure {
            echo "❌ Build failed — check Jenkins console logs"
        }
    }
}
