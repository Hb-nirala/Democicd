pipeline {
    agent any

    parameters {
        choice(
            name: 'BUILDTYPE',
            choices: ['debug-apk', 'release-apk', 'release-aab'],
            description: 'Select Android build type'
        )
        string(
            name: 'KEYSTORE_PASSWORD_CREDENTIAL_ID',
            defaultValue: 'democicd',
            description: 'Jenkins Secret Text credential ID containing the keystore/key password'
        )
        password(
            name: 'KEYSTORE_PASSWORD',
            defaultValue: 'democicd',
            description: 'Optional: provide keystore/key password directly (masked). If set, Jenkins credentials are not used.'
        )
    }

    environment {
        ANDROID_HOME = "/Users/hb/Library/Android/sdk"
        ANDROID_SDK_ROOT = "/Users/hb/Library/Android/sdk"
        JAVA_HOME    = "/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home"
        NODE_BIN = "/Users/hb/.nvm/versions/node/v22.20.0/bin"
        PATH = "${NODE_BIN}:${env.PATH}"
        GRADLE_USER_HOME = "${WORKSPACE}/.gradle"

        MYAPP_RELEASE_STORE_FILE = "${WORKSPACE}/android/app/my-release-key.keystore"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Keystore') {
            when {
                expression { params.BUILDTYPE != 'debug-apk' }
            }
            steps {
                withCredentials([
                file(credentialsId: 'android-keystore', variable: 'MYAPP_RELEASE_STORE_FILE'),
                string(credentialsId: 'keystore-password', variable: 'MYAPP_RELEASE_STORE_PASSWORD'),
                string(credentialsId: 'key-password', variable: 'MYAPP_RELEASE_KEY_PASSWORD'),
                string(credentialsId: 'key-alias', variable: 'MYAPP_RELEASE_KEY_ALIAS')
                ]) {
                    sh '''
                    echo "Preparing keystore..."
                    chmod -R u+w android/app
                    rm -f android/app/my-release-key.keystore
                    cp "$MYAPP_RELEASE_STORE_FILE" android/app/my-release-key.keystore
                    '''
                }
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

        stage('Generate JS Bundle (Release)') {
            when {
                expression { params.BUILDTYPE != 'debug-apk' }
            }
            steps {
                sh '''
                mkdir -p android/app/src/main/assets

                npx react-native bundle \
                  --platform android \
                  --dev false \
                  --entry-file index.js \
                  --bundle-output android/app/src/main/assets/index.android.bundle \
                  --assets-dest android/app/src/main/res
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

        stage('Build Debug APK') {
            when {
                expression { params.BUILDTYPE == 'debug-apk' }
            }
            steps {
                sh '''
                cd android
                ./gradlew assembleDebug
                '''
            }
        }
        
        stage('Build Release APK') {
            when {
                expression { params.BUILDTYPE == 'release-apk' }
            }
            steps {
                
                sh '''
                cd android
                ./gradlew assembleRelease
                '''
            }
        }

        stage('Build Release AAB') {
            when {
                expression { params.BUILDTYPE == 'release-aab' }
            }
            steps {
                sh '''
                cd android
                ./gradlew bundleRelease
                '''
            }
        }

        stage('Archive Android Artifacts') {
            when {
                anyOf {
                    expression { params.BUILDTYPE == 'debug-apk' }
                    expression { params.BUILDTYPE == 'release-apk' }
                    expression { params.BUILDTYPE == 'release-aab' }
                }
            }
            steps {
                archiveArtifacts(
                    allowEmptyArchive: true,
                    fingerprint: true,
                    artifacts: 'android/app/build/outputs/apk/**/*.apk,android/app/build/outputs/bundle/**/*.aab,android/**/build/reports/**'
                )
            }
        }
    }

    post {
        success {
            mail(
                subject: "✅ APK Build Success - ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                    <h2>✅ React Native APK Build Successful</h2>

                    <p><b>Job:</b> ${JOB_NAME}</p>
                    <p><b>Build:</b> #${BUILD_NUMBER}</p>
                    <p><b>Status:</b> ${currentBuild.currentResult}</p>

                    <p>You can view the build details here:</p>
                    <a href="${BUILD_URL}">${BUILD_URL}</a>
                    <p><b>APK file is attached with this email.</b></p>
                """,
                to: "niralak025@gmail.com",
                mimeType: "text/html",
                attachmentsPattern: """
                android/app/build/outputs/**/*.apk,
                android/app/build/outputs/**/*.aab
                """
            )
        }
        failure {
            mail(
                subject: "❌ APK Build FAILED - ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                    <h2>❌ React Native APK Build FAILED</h2>

                    <p><b>Job:</b> ${JOB_NAME}</p>
                    <p><b>Build:</b> #${BUILD_NUMBER}</p>
                    <p><b>Status:</b> ${currentBuild.currentResult}</p>

                    <p>Check the console logs:</p>
                    <a href="${BUILD_URL}console">${BUILD_URL}console</a>
                """,
                to: "niralak025@gmail.com",
                mimeType: "text/html"
            )
        }
    }
}
