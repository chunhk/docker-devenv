FROM ubuntu:bionic
MAINTAINER Chun (chun@localhost)
WORKDIR /workspace

# Install baee packages
RUN export DEBIAN_FRONTEND=noninteractive \
		&& apt-get update \
		&& apt-get install -y curl wget less software-properties-common \
		apt-utils p7zip unzip zip tmux ntp pkg-config zlib1g-dev sudo vim \
		apt-transport-https
RUN ln -fs /usr/share/zoneinfo/America/Los_Angeles /etc/localtime \
		&& dpkg-reconfigure --frontend noninteractive tzdata

# Install Python
RUN apt-get -y install python python3-dev python3-pip python-dev \
		python-pip pyflakes
# Final yapf will overwrite others. If you want to run yapf of a different
# version, just change /usr/local/bin/yapf "#/usr/bin/python3"
RUN pip install futures && pip install yapf && pip3 install yapf

# Setup a non-root dev user
# Note setting the passowrd below is not secure, do not use for production
RUN adduser --shell /bin/bash --gecos "" --disabled-password devuser \
		&& adduser devuser sudo \
		&& echo 'devuser:devuser' | chpasswd

RUN apt-get clean

# -- APT REPOSITORIES  -- 

# Oracle Java 8
RUN add-apt-repository -y ppa:webupd8team/java

# Oracle Java 10 
RUN add-apt-repository -y ppa:linuxuprising/java

# Unofficial Go
# Note: You may have to find another repo when new versions are released
RUN add-apt-repository -y ppa:hnakamur/golang-1.10

# Bazel
RUN echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list
RUN curl https://bazel.build/bazel-release.pub.gpg | apt-key add -

# Erlang and Elixir
RUN wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb \
		&& dpkg -i erlang-solutions_1.0_all.deb

# R
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9

# Crystal - This doesn't work, so we are using deb pkg below instead.
# RUN curl https://dist.crystal-lang.org/apt/setup.sh | bash

# Node.js
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash

# Dart
RUN sh -c 'curl https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -'
RUN sh -c 'curl https://storage.googleapis.com/download.dartlang.org/linux/debian/dart_stable.list > /etc/apt/sources.list.d/dart_stable.list'

# Refresh Apt Cache after repositories added
RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get clean

# -- Install Packages --

# Install Python
RUN apt-get install python3-dev python3-pip python-dev python-pip pyflakes
RUN pip install futures && pip install yapf && pip3 install yapf

# Install Git
RUN apt-get install -y git git-flow

# Install C/C++ Compilers
RUN apt-get install -y gcc g++

# Install build tools
RUN apt-get install -y build-essential cmake exuberant-ctags

# Install Crystal
RUN apt-get install -y libpcre3-dev libevent-dev \
		&& wget https://github.com/crystal-lang/crystal/releases/download/0.25.1/crystal_0.25.1-1_amd64.deb \
		&& dpkg -i crystal_0.25.1-1_amd64.deb

# Accept Oracle Java License
RUN echo debconf shared/accepted-oracle-license-v1-1 select true | debconf-set-selections
RUN echo debconf shared/accepted-oracle-license-v1-1 seen true | debconf-set-selections

# Install Oracle Java8 JDK
RUN add-apt-repository -y ppa:webupd8team/java \
		&& apt-get update \
		&& apt-get install -y oracle-java8-installer

# Install Oracle Java10 JDK
RUN add-apt-repository -y ppa:linuxuprising/java \
		&& apt-get update \
		&& apt-get install -y oracle-java10-installer

# Install Apache Maven
RUN wget http://mirrors.ocf.berkeley.edu/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz \
		&& mkdir -p /usr/local/java_lib/apache-maven \
		&& tar -xf apache-maven-3.5.4-bin.tar.gz -C /usr/local/java_lib/apache-maven --strip-components=1 \
		&& ln -s /usr/local/java_lib/apache-maven/bin/mvn /usr/local/bin/

# Install Bazel
RUN apt-get install -y bazel

# Install Erlang/OTP and Elixir
RUN apt-get install -y esl-erlang elixir

# Install go-lang
RUN apt-get install -y golang-go golang-1.10-doc

# Install Haskell
RUN apt-get install -y ghc haskell-platform haskell-stack cabal-install

# Install PyTorch
RUN pip3 install http://download.pytorch.org/whl/cpu/torch-0.4.0-cp36-cp36m-linux_x86_64.whl \
		&& pip3 install torchvision

# Install Tensorflow
RUN pip3 install -U tensorflow

# Install Jupyter and Other Data Science Libraries
# Start: jupyter notebook --port 8080 --ip 0.0.0.0
RUN pip3 install jupyter numpy scipy pandas statsmodels numexpr cython tables scikit-learn matplotlib

# Install R
RUN apt-get install -y r-base r-base-dev r-recommended

# Install R Studio
RUN apt-get install -y gdebi-core psmisc \
		&& wget https://download2.rstudio.org/rstudio-server-1.1.453-amd64.deb \
		&& gdebi -n rstudio-server-1.1.453-amd64.deb \
		&& echo 'server-app-armor-enabled=0' >> /etc/rstudio/rserver.conf

# Install Node.js
RUN apt-get install -y nodejs
RUN npm install js-beautify

# Install Dart
RUN apt-get install -y dart

RUN apt-get clean

# Switch to non root user. Hereafter, commands are run under the specified user
USER devuser

# Install Rust, need to run % source $HOME/.cargo/env
RUN curl https://sh.rustup.rs -sSf | sh -s -- --default-toolchain stable -y -v
RUN echo "export PATH=$PATH:$HOME/.cargo/bin" >> $HOME/.bashrc

# -- VIM SETUP --

# Install NeoBundle
RUN curl https://raw.githubusercontent.com/Shougo/neobundle.vim/master/bin/install.sh | sh

# Use vimrc_pre to setup base environment, so that vim can start without prompts
COPY vimrc_pre .
RUN cp vimrc_pre $HOME/.vimrc
RUN vim +NeoBundleInstall +qall --not-a-term

# Build YouCompleteMe
RUN export PATH=$PATH:$HOME/.cargo/bin && cd $HOME/.vim/bundle/YouCompleteMe && ./install.py --clang-completer --go-completer --js-completer --rust-completer --java-completer

# Copy Java Formatter
RUN mkdir $HOME/lib && cd $HOME/lib && wget https://github.com/google/google-java-format/releases/download/google-java-format-1.6/google-java-format-1.6-all-deps.jar

# Copy final vimrc
COPY vimrc .
RUN cp vimrc $HOME/.vimrc

# Copy tmux conf
COPY tmux.conf .
RUN cp tmux.conf $HOME/.tmux.conf
