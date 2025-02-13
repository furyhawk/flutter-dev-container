#-------------------------------------------------------------------------------------------------------------
# Flutter Dev Container - Lucas Hilleshein dos Santos.
# Licensed under the MIT License.
# See https://github.com/lucashilles/flutter-dev-container/blob/master/LICENSE for license information.
#-------------------------------------------------------------------------------------------------------------

FROM ubuntu:latest

#Locale
ENV LANG C.UTF-8



# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

#
# Install needed packages, setup user and clean up.
RUN  apt update \
	&& apt install -y sudo \
	&& apt install -y openjdk-17-jdk-headless --no-install-recommends \
	&& apt install -y wget curl git xz-utils zip unzip --no-install-recommends \
	# Clean Up
	&& apt-get autoremove -y \
	&& apt-get clean -y \
	&& rm -rf /var/lib/apt/lists/* 
	
	# Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
	# [Optional] Add sudo support for the non-root user
RUN if ! getent group $USER_GID; then groupadd --gid $USER_GID $USERNAME; fi \
	&& if id -u $USER_UID >/dev/null 2>&1; then USER_UID=$(($USER_UID + 1)); fi \
	&& useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
	&& echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
	&& chmod 0440 /etc/sudoers.d/$USERNAME \
	&& su $USERNAME \
	&& cd $HOME



#
# Android SDK
# https://developer.android.com/studio#downloads
ENV ANDROID_SDK_TOOLS_VERSION=11076708
ENV ANDROID_PLATFORM_VERSION=34
ENV ANDROID_BUILD_TOOLS_VERSION=34.0.0
ENV ANDROID_HOME=/opt/android-sdk-linux
ENV ANDROID_SDK_ROOT="$ANDROID_HOME"
ENV PATH=${PATH}:${ANDROID_HOME}/cmdline-tools/cmdline-tools/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/emulator

#
# Flutter SDK
# https://flutter.dev/docs/development/tools/sdk/releases?tab=linux
ENV FLUTTER_CHANNEL="stable"
ENV FLUTTER_VERSION="3.19.3"
# Make sure to use the needed channel and version for this.
ENV FLUTTER_HOME=/opt/flutter
ENV PATH=${PATH}:${FLUTTER_HOME}/bin

# Download Android and Flutter
RUN curl -C - --output android-sdk-tools.zip https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_SDK_TOOLS_VERSION}_latest.zip
RUN curl -C - --output flutter.tar.xz https://storage.googleapis.com/flutter_infra_release/releases/${FLUTTER_CHANNEL}/linux/flutter_linux_${FLUTTER_VERSION}-${FLUTTER_CHANNEL}.tar.xz 

# Install Android SDK	
RUN mkdir -p ${ANDROID_HOME}/cmdline-tools/latest \
	&& unzip -q android-sdk-tools.zip -d ${ANDROID_HOME}/cmdline-tools/latest \
	&& yes | ${ANDROID_HOME}/cmdline-tools/latest/bin/sdkmanager --licenses \
	&& touch $HOME/.android/repositories.cfg \
	&& ${ANDROID_HOME}/cmdline-tools/latest/bin/sdkmanager platform-tools \
	&& ${ANDROID_HOME}/cmdline-tools/latest/bin/sdkmanager emulator \
	&& ${ANDROID_HOME}/cmdline-tools/latest/bin/sdkmanager "platforms;android-${ANDROID_PLATFORM_VERSION}" "build-tools;$ANDROID_BUILD_TOOLS_VERSION" \
	&& ${ANDROID_HOME}/cmdline-tools/latest/bin/sdkmanager --install "cmdline-tools;latest" 

# Create Android Emulator	
RUN ${ANDROID_HOME}/cmdline-tools/latest/bin/sdkmanager "system-images;android-${ANDROID_PLATFORM_VERSION};google_apis;x86_64" \
	&& echo "no" | ${ANDROID_HOME}/cmdline-tools/latest/bin/avdmanager create avd -n Android${ANDROID_PLATFORM_VERSION} -k "system-images;android-${ANDROID_PLATFORM_VERSION};google_apis;x86_64"


# Install Flutter SDK
RUN tar -xf flutter.tar.xz -C /opt/ \
	&& git config --global --add safe.directory /opt/flutter \
	&& flutter config --android-sdk "${ANDROID_SDK_ROOT}" \
	&& yes | flutter doctor --android-licenses \
	&& flutter config --no-analytics \
	&& flutter update-packages

# Clean up
RUN rm flutter.tar.xz \
	&& rm android-sdk-tools.zip
