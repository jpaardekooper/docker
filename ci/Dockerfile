FROM ubuntu:focal

# create jenkins user
ARG JENKINS_UID=999
RUN (id ${JENKINS_UID} && userdel $(id -un ${JENKINS_UID}) || true) && adduser --disabled-password --gecos "" --uid ${JENKINS_UID} --gid 1 jenkins

# Allow kvm/emulator without root.
# Note: group id 117 is host dependant.
RUN groupadd -r kvm -g 117 && gpasswd -a jenkins kvm

# General config

ENV ANDROID_HOME=/home/jenkins/sdks/android \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LANGUAGE=en_US:en

ENV ANDROID_SDK_ROOT=$ANDROID_HOME \
    PATH=${PATH}:${ANDROID_HOME}/cmdline-tools/tools/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/emulator

# Install Android SDK
# sources:
# - https://github.com/cirruslabs/docker-images-android/blob/fc1ba6091b9411226dede37c76fd589492c2469a/sdk/tools/Dockerfile
# - https://github.com/cirruslabs/docker-images-android/blob/fc1ba6091b9411226dede37c76fd589492c2469a/sdk/30/Dockerfile

USER root

# comes from https://developer.android.com/studio/#command-tools
ENV ANDROID_SDK_TOOLS_VERSION 6609375

RUN set -o xtrace \
    && apt-get update \
    && apt-get install -y openjdk-8-jdk \
    && apt-get install -y wget zip unzip git curl bc software-properties-common build-essential lib32stdc++6 libstdc++6 libpulse0 libglu1-mesa locales lcov libsqlite3-0 --no-install-recommends \
    # for x86 emulators
    && apt-get install -y libxtst6 libnss3-dev libnspr4 libxss1 libasound2 libatk-bridge2.0-0 libgtk-3-0 libgdk-pixbuf2.0-0 \
    && rm -rf /var/lib/apt/lists/* \
    && sh -c 'echo "en_US.UTF-8 UTF-8" > /etc/locale.gen' \
    && locale-gen \
    && update-locale LANG=en_US.UTF-8 \
    && apt-get clean \
    && rm -rf /var/lib/{apt,dpkg,cache,log} /tmp/* /var/tmp/*

# Install appcenter-cli
RUN apt-get update \
  && apt-get -y install \
  curl gpg-agent \
  && curl -sL https://deb.nodesource.com/setup_14.x | bash - \
  && apt-get install --yes nodejs \
  && npm install -g appcenter-cli \
  && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

USER jenkins

# Base Android SDK installation
RUN wget -q https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_SDK_TOOLS_VERSION}_latest.zip -O /tmp/android-sdk-tools.zip \
    && mkdir -p ${ANDROID_HOME}/cmdline-tools/ \
    && unzip -q /tmp/android-sdk-tools.zip -d ${ANDROID_HOME}/cmdline-tools/ \
    && rm /tmp/android-sdk-tools.zip \
    && yes | sdkmanager --licenses \
    && mkdir -p /home/jenkins/.android \
    && touch /home/jenkins/.android/repositories.cfg \
    && sdkmanager platform-tools

ENV ANDROID_PLATFORM_VERSION 29
ENV ANDROID_BUILD_TOOLS_VERSION 28.0.3

# Project-specific SDK dependencies
RUN yes | sdkmanager \
    # specify all additional required android platforms here
    "platforms;android-28" \
    "platforms;android-$ANDROID_PLATFORM_VERSION" \
    "build-tools;$ANDROID_BUILD_TOOLS_VERSION" \
    && sdkmanager "system-images;android-$ANDROID_PLATFORM_VERSION;google_apis;x86_64" \
    && echo no | avdmanager create avd --force -n android-emulator -k "system-images;android-$ANDROID_PLATFORM_VERSION;google_apis;x86_64"

# Install Flutter
# Source: https://github.com/cirruslabs/docker-images-flutter/blob/96b86c161bbf204ffd11f8590e08726d796e8cb1/sdk/Dockerfile

# TODO: specify Flutter version here 👇
ENV FLUTTER_VERSION=2.0.2
ENV FLUTTER_HOME=/home/jenkins/sdks/flutter
ENV FLUTTER_ROOT=$FLUTTER_HOME

ENV PATH ${PATH}:${FLUTTER_HOME}/bin:${FLUTTER_HOME}/bin/cache/dart-sdk/bin

RUN git clone --branch ${FLUTTER_VERSION} https://github.com/flutter/flutter.git ${FLUTTER_HOME} \
    && yes | flutter doctor --android-licenses \
    && flutter doctor

# Pre-cache Dart dependencies
COPY --chown=jenkins:root pubspec.lock pubspec.yaml /tmp/
RUN cd /tmp && dart pub get