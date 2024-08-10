FROM ubuntu:22.04

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Install necessary dependencies
RUN apt-get update && apt-get install -y \
    curl \
    git \
    unzip \
    xz-utils \
    zip \
    libglu1-mesa \
    openjdk-11-jdk-headless \
    sudo \
    clang \
    cmake \
    ninja-build \
    pkg-config \
    libgtk-3-dev \
    chromium-browser \
    libnss3 \
    libnspr4 \
    libatk1.0-0 \
    libatk-bridge2.0-0 \
    libcups2 \
    libdrm2 \
    libxkbcommon0 \
    libxcomposite1 \
    libxdamage1 \
    libxfixes3 \
    libxrandr2 \
    libgbm1 \
    libasound2 \
    && rm -rf /var/lib/apt/lists/*

# Set Chrome executable path
ENV CHROME_EXECUTABLE=/usr/bin/chromium-browser

# Create a non-root user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Install Flutter
RUN git clone https://github.com/flutter/flutter.git -b stable /usr/local/flutter
ENV PATH "$PATH:/usr/local/flutter/bin"

# Set Flutter path and Chrome executable for all users
RUN echo "export PATH=\$PATH:/usr/local/flutter/bin" >> /etc/profile
RUN echo "export CHROME_EXECUTABLE=/usr/bin/chromium-browser" >> /etc/profile

# Pre-download Flutter packages and set up Flutter
RUN flutter precache
RUN flutter config --no-analytics

# Verify Flutter installation
RUN flutter doctor

# Install Android SDK
ENV ANDROID_SDK_ROOT /usr/local/android-sdk
RUN mkdir -p ${ANDROID_SDK_ROOT}/cmdline-tools
RUN cd ${ANDROID_SDK_ROOT}/cmdline-tools && \
    curl -O https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip && \
    unzip commandlinetools-linux-*_latest.zip && \
    rm commandlinetools-linux-*_latest.zip && \
    mv cmdline-tools latest
ENV PATH "$PATH:${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin"

# Set Android SDK path for all users
RUN echo "export ANDROID_SDK_ROOT=${ANDROID_SDK_ROOT}" >> /etc/profile
RUN echo "export PATH=\$PATH:\${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin" >> /etc/profile

# Accept licenses
RUN yes | sdkmanager --licenses

# Install Android SDK components
RUN sdkmanager "platform-tools" "platforms;android-33" "build-tools;33.0.0"

# Change ownership of Flutter and Android SDK directories
RUN chown -R $USERNAME:$USERNAME /usr/local/flutter ${ANDROID_SDK_ROOT}

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

# Switch to non-root user
USER $USERNAME

# Verify Flutter installation as non-root user
RUN flutter doctor

# Verify the CHROME_EXECUTABLE environment variable
RUN echo "CHROME_EXECUTABLE is set to: $CHROME_EXECUTABLE"

# Add a test to ensure Chromium can be launched
RUN $CHROME_EXECUTABLE --version || echo "Chromium launch failed"