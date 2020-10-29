FROM ubuntu:20.04

LABEL maintainer="Anton Malinskiy anton@malinskiy.com"

# Set up insecure default key
COPY adbkey adbkey.pub adb_usb.ini $HOME/.android/

ENV LINK_ANDROID_SDK=https://dl.google.com/android/repository/commandlinetools-linux-6858069_latest.zip \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8 \
    ANDROID_HOME=/opt/android-sdk-linux \
    ANDROID_CMD_ROOT=/opt/cmdline-tools \
    DEBIAN_FRONTEND=noninteractive \
    PATH="$PATH:/opt/android-sdk-linux/tools:/opt/android-sdk-linux/platform-tools:/opt/android-sdk-linux/tools/bin:/opt/android-sdk-linux/emulator:$ANDROID_CMD_ROOT/cmdline-tools:$ANDROID_CMD_ROOT/bin"

RUN dpkg --add-architecture i386 && \
    apt-get update && \
    apt-get install -yq software-properties-common libstdc++6:i386 zlib1g:i386 libncurses5:i386 \
        locales ca-certificates apt-transport-https curl unzip redir iproute2 \
        openjdk-8-jdk xvfb x11vnc fluxbox nano telnet expect \
        libc6 libdbus-1-3 libfontconfig1 libgcc1 \
        libpulse0 libtinfo5 libx11-6 libxcb1 libxdamage1 \
        libnss3 libxcomposite1 libxcursor1 libxi6 \
        libxext6 libxfixes3 zlib1g libgl1 pulseaudio socat \
        --no-install-recommends && \
        locale-gen en_US.UTF-8 && \
    # Install Android SDK
    curl -L $LINK_ANDROID_SDK > /tmp/android-sdk-linux.zip && \
    unzip -q /tmp/android-sdk-linux.zip -d $ANDROID_CMD_ROOT && \
    rm /tmp/android-sdk-linux.zip && \
    # Customized steps per specific platform
    mkdir $HOME/.android && \
    touch $HOME/.android/repositories.cfg && \
    yes | $ANDROID_CMD_ROOT/cmdline-tools/bin/sdkmanager --sdk_root="$ANDROID_HOME" --licenses && \
    yes | $ANDROID_CMD_ROOT/cmdline-tools/bin/sdkmanager --sdk_root="$ANDROID_HOME" "platforms;{{ platform }}" "tools" "system-images;{{ platform }};google_apis;x86_64" --verbose && \
    echo no | avdmanager create avd -n "Pixel2" --package "system-images;{{ platform }};google_apis;x86_64" --tag google_apis && \
    apt-get purge -yq unzip openjdk-8-jdk && \
    # Clean up
    apt-get -yq autoremove && \
    apt-get clean && \
    apt-get autoclean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY config.ini /root/.android/avd/Pixel2.avd/config.ini

# Expose adb
EXPOSE 5037 5554 5555 5900

HEALTHCHECK --interval=2s --timeout=40s --retries=1 \
    CMD timeout 40 adb wait-for-device shell 'while [[ -z $(getprop sys.boot_completed) ]]; do sleep 1; done'

# Add script
COPY start.sh /start.sh
RUN chmod +x /start.sh
CMD ["/start.sh"]
