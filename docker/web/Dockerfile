FROM amazonlinux:2 as installer

# awscli_v2をインストール
RUN set -x \
    && yum install -y unzip \
    && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
    && unzip awscliv2.zip \
    && ./aws/install -i /usr/local/aws-cli -b /usr/local/bin/aws

FROM amazonlinux:2
LABEL maintainer="<infratop>" \
    description="Amazon Linux 2 with some development environments"

# パッケージのバージョンを設定
ENV RUBY_VERSION="3.1.2" \
    RAILS_VERSION="6.1.4" \
    NOKOGIRI_VERSION="1.14.1" \
    YARN_VERSION="1.22.11" \
    PYTHON_VERSION="3.9.10" \
    TERRAFORM_VERSION="1.1.5" \
    ANSIBLE_VERSION="2.9.0" \
    USERNAME="ec2-user" \
    PASSWORD=""

# パッケージをインストール
RUN set -x \
    && yum update -y && yum install -y \
    bzip2 \
    bzip2-devel \
    curl \
    gcc-c++ \
    git \
    jq \
    less \
    libffi-devel \
    make \
    openssl \
    openssl-devel \
    perl-Digest-SHA \
    procps \
    readline \
    readline-devel \
    sqlite-devel \
    sudo \
    tar \
    mysql \
    unzip \
    wget \
    yum-utils \
    zlib-devel \
    keybase \
    && amazon-linux-extras install epel -y \
    && yum install -y sshpass \
    && yum clean all \
    && rm -rf /var/cache/yum

# ec2-userを追加
RUN useradd -m -r -G wheel -s /bin/bash ${USERNAME} \
    && echo "${USERNAME}:${PASSWORD}" | chpasswd \
    && echo '%wheel ALL=(ALL) ALL' | EDITOR='tee -a' visudo \
    && echo "${USERNAME}   ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

COPY --from=installer /usr/local/aws-cli/ /usr/local/aws-cli/
COPY --from=installer /usr/local/bin/ /usr/local/bin/

USER ${USERNAME}
WORKDIR /home/${USERNAME}/environment

# tfenvをインストール
RUN git clone https://github.com/tfutils/tfenv.git ~/.tfenv \
    && echo 'export PATH="$HOME/.tfenv/bin:$PATH"' | tee -a ~/.bash_profile \
    # pyenvをインストール
    && git clone https://github.com/pyenv/pyenv.git ~/.pyenv \
    && echo 'export PATH="$HOME/.pyenv/shims:$HOME/.pyenv/bin:$PATH"' | tee -a ~/.bash_profile \
    && echo 'eval "$(pyenv init -)"' | tee -a ~/.bash_profile \
    # rbenvをインストール
    && git clone https://github.com/sstephenson/rbenv.git ~/.rbenv \
    && git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build \
    && echo 'export PATH="$HOME/.rbenv/bin:$PATH"' | tee -a ~/.bash_profile \
    && echo 'eval "$(rbenv init -)"' | tee -a ~/.bash_profile \
    && echo 'export PATH="/usr/local/bin/aws:$PATH"' | tee -a ~/.bash_profile \
    && . ~/.bash_profile \
    # Terraformをインストール
    && tfenv install ${TERRAFORM_VERSION} \
    && tfenv use ${TERRAFORM_VERSION} \
    # Pythonをインストール
    && pyenv install ${PYTHON_VERSION} \
    && pyenv global ${PYTHON_VERSION} \
    # Rubyをインストール
    && rbenv install ${RUBY_VERSION} \
    && rbenv global ${RUBY_VERSION} \
    # pipをアップグレード
    && pip3 install --upgrade pip \
    # wheelをインストール
    && pip3 install wheel \
    # Ansibleをインストール
    && pip3 install ansible~=${ANSIBLE_VERSION} \
    # nodeをインストール
    && curl --silent --location https://rpm.nodesource.com/setup_16.x | sudo bash - \
    && sudo yum install -y nodejs \
    # yarnをインストール
    && curl -L --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz" > /tmp/yarn.tar.gz \ 
    && sudo tar -xzf /tmp/yarn.tar.gz -C /opt && \
    sudo ln -s /opt/yarn-v$YARN_VERSION/bin/yarn /usr/local/bin/yarn && \
    sudo ln -s /opt/yarn-v$YARN_VERSION/bin/yarnpkg /usr/local/bin/yarnpkg && \
    sudo rm /tmp/yarn.tar.gz \
    # Railsをインストール
    && gem install nokogiri -v ${NOKOGIRI_VERSION} \
    && gem install rails -v ${RAILS_VERSION}



