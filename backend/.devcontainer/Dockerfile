# CUDA 12.1.1の開発ベースイメージを使用
FROM nvidia/cuda:12.1.1-devel-ubuntu22.04

# 環境変数で非対話的な設定を強制
ENV DEBIAN_FRONTEND=noninteractive

# 必要なパッケージのインストールとロケールの設定
RUN apt-get update && apt-get install -y \
    locales \
    wget \
    git \
    curl \
    build-essential \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    llvm \
    libncurses5-dev \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libffi-dev \
    liblzma-dev \
    tzdata \
    sudo && \
    locale-gen ja_JP.UTF-8 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# タイムゾーンの設定
ENV TZ=Asia/Tokyo

# Python 3.11.0のインストール
RUN wget https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz && \
    tar xzf Python-3.11.0.tgz && \
    cd Python-3.11.0 && \
    ./configure --enable-optimizations && \
    make altinstall && \
    cd .. && \
    rm -rf Python-3.11.0 Python-3.11.0.tgz

# Pythonのバージョンを3.11.0に設定
RUN update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.11 1 && \
    update-alternatives --set python3 /usr/local/bin/python3.11 && \
    update-alternatives --install /usr/bin/python python /usr/local/bin/python3.11 1 && \
    update-alternatives --set python /usr/local/bin/python3.11

# pipのアップグレード
RUN curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    python get-pip.py && \
    rm get-pip.py

# vscodeユーザーの作成
RUN useradd -m -s /bin/bash -G sudo vscode && \
    echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/vscode

# 環境変数をリセット（非対話的な設定を解除)
ENV DEBIAN_FRONTEND=

# 環境変数の設定
ENV LANG ja_JP.UTF-8
ENV LANGUAGE ja_JP:ja
ENV LC_ALL ja_JP.UTF-8
ENV TZ JST-9
ENV TERM xterm

# CUDAライブラリへのパスを設定
ENV PATH=/usr/local/cuda/bin:${PATH}
ENV LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/local/cuda/compat:${LD_LIBRARY_PATH}

# requirements.txtからパッケージをインストール
COPY requirements.txt /root/backend/requirements.txt
RUN pip install -r /root/backend/requirements.txt

# CUDA対応のllama-cpp-pythonを別途インストール
RUN pip install llama-cpp-python --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cu124 --upgrade --force-reinstall --no-cache-dir

# Pythonのバージョンを確認
RUN python --version

# vscodeユーザーに切り替え
USER vscode

# ワークディレクトリの設定
WORKDIR /home/vscode