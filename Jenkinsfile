// TODO: specify project id here 👇
projectId = "Flutter-Happines-POC-1"

pipeline {
  agent {
    dockerfile {
      filename "ci/Dockerfile"
      additionalBuildArgs "${support.ciDockerFileBuildArgs()} --pull"
      args "${support.ciDockerFileRunArgs(projectId)} -u jenkins --device=/dev/kvm"
    }
  }

  options {
    ansiColor("xterm") // provides coloring of console output
    buildDiscarder(logRotator(numToKeepStr: "10", daysToKeepStr: "3")) // configures job cleanup
  }

  environment {
    GRADLE_USER_HOME = "${WORKSPACE}/tmp/gradle"
  }

  stages {
    stage("Setup") {
      steps {
        sh "flutter pub get"
        sh "mkdir -p  '${GRADLE_USER_HOME}'"
      }
    }

    stage("Tests") {
      steps {
        sh "git ls-files '**/*.dart' | xargs flutter format -n --set-exit-if-changed"
        sh "flutter test"
        sh "flutter analyze"
      }
    }

    // stage("Integration Tests") {
    //   steps {
    //     sh "\$ANDROID_HOME/emulator/emulator -avd android-emulator -no-audio -no-window -no-snapshot -no-boot-anim &"
    //     sh "adb wait-for-device shell 'while [[ -z \$(getprop sys.boot_completed) ]]; do sleep 1; done; input keyevent 82'"
    //     sh "flutter drive --target test_driver/app.dart"
    //   }
    // }

    stage("Development build") {
      when { branch "master" }

      environment {
        // TODO: Create an API token in AppCenter, configure it in Jenkins project credentials
        APPCENTER_TOKEN = credentials("")
      }

      steps {
        /* Caching build/ across multiple builds sometimes leads to builds with libflutter.so missing ¯\_(ツ)_/¯ */
        sh "rm -rf build/"

        sh """
        flutter build apk --release \
          --target-platform android-arm,android-arm64 \
          --split-per-abi
        """

        // TODO: specify AppCenter project ID here 👇
        sh "appcenter distribute release -f build/app/outputs/apk/release/app-arm64-v8a-release.apk --app 'Flutter-Happines-POC-1' -g Collaborators --token ${APPCENTER_TOKEN}"
        archiveArtifacts artifacts: 'build/app/outputs/apk/release/*.apk', onlyIfSuccessful: true
      }
    }

    stage("Production build") {
      when { branch "release/*" }

      environment {
        APPCENTER_TOKEN = credentials("APPCENTER_TOKEN_PROD")
      }

      steps {
        /* Caching build/ across multiple builds sometimes leads to builds with libflutter.so missing ¯\_(ツ)_/¯ */
        sh "rm -rf build/"

        sh """
        flutter build apk --target lib/main_prod.dart \
          --release \
          --target-platform android-arm,android-arm64 \
          --split-per-abi
        """

        // TODO: specify AppCenter project ID here 👇
        sh "appcenter distribute release -f build/app/outputs/apk/release/app-arm64-v8a-release.apk --app 'TODO-AppCenter-project-ID-here' -g Collaborators --token ${APPCENTER_TOKEN}"
        archiveArtifacts artifacts: 'build/app/outputs/apk/release/*.apk', onlyIfSuccessful: true
      }
    }
  }
}