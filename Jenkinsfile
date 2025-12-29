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
        GRADLE_USER_HOME = "${WORKSPACE}/.gradle"

        GDRIVE_BIN = "/usr/local/bin"

        PATH = "${NODE_BIN}:${GDRIVE_BIN}:${env.PATH}"

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

        stage('Verify gdrive') { 
            steps { 
                sh ''' 
                    which gdrive || true 
                    gdrive version || true 
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

        stage('Upload APK to Google Drive') {
            steps {
                sh '''
                echo "Finding APK..."
                APK_PATH=$(ls android/app/build/outputs/apk/**/*.apk | head -n 1)

                if [ -z "$APK_PATH" ]; then
                    echo "❌ APK not found"
                    exit 1
                fi

                APK_NAME=$(basename "$APK_PATH")

                echo "Uploading APK to Google Drive..."
                echo "APK Path : $APK_PATH"
                echo "APK Name : $APK_NAME"

                FILE_ID=$(gdrive files upload \
                --parent 1gs78v2H82xwdw-WfSmpwwpcpuprb7qyo \
                "$APK_PATH")

                echo "FILE_ID=$FILE_ID" > apk_info.txt
                echo "APK uploaded successfully with ID: $FILE_ID"
                '''
            }
        }

        stage('Generate APK Download Link') {
            steps {
                sh '''
                source apk_info.txt
                APK_LINK="https://drive.google.com/file/d/$FILE_ID/view?usp=sharing"
                echo "APK_LINK=$APK_LINK" > apk_link.txt
                '''
            }
        }
    }

    post {
        success {
            script {
                def apkLink = sh(
                script: "source apk_link.txt && echo \$APK_LINK",
                returnStdout: true
                ).trim()
            echo "APK Download Link: ${apkLink}"
            echo "Sending success email with APK link..."
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
                        <p><b>Download APK:</b></p>
                        <p><a href="${apkLink}">${apkLink}</a></p>
                    """,
                    to: "niralak025@gmail.com",
                    mimeType: "text/html",
                )
            }
        }
        failure {
            echo "APK build failed. Sending failure email."
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
