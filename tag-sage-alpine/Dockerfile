FROM alpine

LABEL sh.demyx.image demyx/code-server:sage
LABEL sh.demyx.maintainer Demyx <info@demyx.sh>
LABEL sh.demyx.url https://demyx.sh
LABEL sh.demyx.github https://github.com/demyxco
LABEL sh.demyx.registry https://hub.docker.com/u/demyx

# Packages
RUN set -ex; \
    apk add --update --no-cache \
    bash \
    curl \
    dumb-init \
    composer \
    gcc \
    git \
    gnupg \
    htop \
    jq \
    mysql-client \
    nano \
    npm \
    openssh \
    php7 \
    php7-ctype \
    php7-json \
    php7-mbstring \
    php7-mysqli \
    php7-phar \
    php7-simplexml \
    php7-tokenizer \
    php7-xmlwriter \
    sudo \
    util-linux \
    tzdata \
    zsh; \
    \
    npm -g install yarn

# wp-cli
RUN set -ex; \
    curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar; \
    chmod +x wp-cli.phar; \
    mv wp-cli.phar /usr/local/bin/wp

# Create and configure demyx user
RUN set -ex; \
    addgroup -g 1000 -S demyx; \
    adduser -u 1000 -D -S -G demyx demyx; \
    \
    # Oh-My-Zsh
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"; \
    git clone https://github.com/zsh-users/zsh-autosuggestions.git /root/.oh-my-zsh/plugins/zsh-autosuggestions; \
    sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="ys"/g' /root/.zshrc; \
    sed -i "s/(git)/(git zsh-autosuggestions)/g" /root/.zshrc; \
    \
    su -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" -s /bin/sh demyx; \
    su -c  "git clone https://github.com/zsh-users/zsh-autosuggestions.git /home/demyx/.oh-my-zsh/plugins/zsh-autosuggestions" -s /bin/sh demyx; \
    sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="ys"/g' /home/demyx/.zshrc; \
    sed -i "s/(git)/(git zsh-autosuggestions)/g" /home/demyx/.zshrc; \
    \
    # Symlink demyx command history with root
    ln -s /home/demyx/.zsh_history /root; \
    \
    # Change shell
    sed -i "s|/root:/bin/bash|/root:/bin/zsh|g" /etc/passwd; \
    sed -i "s|/home/demyx:/sbin/nologin|/home/demyx:/bin/zsh|g" /etc/passwd; \
    \
    # Create code-server directories and set default shell for code-server
    mkdir -p /home/demyx/.code/extensions; \
    mkdir -p /home/demyx/.code/data/User; \
    echo -e "{\n    \"terminal.integrated.shell.linux\": \"/bin/zsh\"\n}" > /home/demyx/.code/data/User/settings.json; \
    \
    # No password when using sudo
    echo "demyx ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/nopasswd; \
    \
    # gpg purposes
    echo 'export GPG_TTY=$(tty)' >> /root/.zshrc; \
    echo 'export GPG_TTY=$(tty)' >> /home/demyx/.zshrc; \
    \
    chown -R demyx:demyx /home/demyx

# Download Alpine binary of code-server
RUN set -ex; \
    export DEMYX_CODER_BINARY=$(curl -sL https://api.github.com/repos/cdr/code-server/releases/latest | grep '"name"' | head -n1 | awk -F '[:]' '{print $2}' | sed -e 's/"//g' | sed -e 's/,//g' | sed -e 's/ //g' | sed -e 's/\r//g'); \
    # Set fixed version as a fallback if curling fails
    if [ -z "$DEMYX_CODER_BINARY" ]; then export DEMYX_CODER_BINARY=2.1650-vsc1.39.2; fi; \
    wget https://github.com/cdr/code-server/releases/download/"$DEMYX_CODER_BINARY"/code-server"$DEMYX_CODER_BINARY"-alpine-x86_64.tar.gz -qO /tmp/code-server"$DEMYX_CODER_BINARY"-alpine-x86_64.tar.gz; \
    tar -xzf /tmp/code-server"$DEMYX_CODER_BINARY"-alpine-x86_64.tar.gz -C /tmp; \
    mv /tmp/code-server"$DEMYX_CODER_BINARY"-alpine-x86_64/code-server /usr/local/bin; \
    rm -rf /tmp/*

# Install code-server Laravel Blade extension
RUN set -ex; \
    su -c 'code-server --extensions-dir /home/demyx/.code/extensions --install-extension cjhowe7.laravel-blade --force' -s /bin/sh demyx

# Install Sage
RUN set ex; \
    su -c 'composer create-project -n roots/sage /tmp/sage; \
        yarn --cwd=/tmp/sage; \
        yarn --cwd=/tmp/sage build; \
        yarn cache clean; \
        composer clearcache' -s /bin/sh demyx; \
    mkdir -p /usr/src; \
    chown demyx:demyx /usr/src; \
    tar -czf /usr/src/sage.tgz -C /tmp sage; \
    rm -rf /tmp/*; \
    chown -R root:root /usr/src

# Copy files
COPY demyx.sh /usr/local/bin/demyx
COPY bs.js /usr/src/bs.js
# Sage helper for yarn
COPY sage.sh /usr/local/bin/sage
# A static homepage will redirect BrowserSync, this plugin disables that.
COPY bs.php /usr/src/bs.php

# Finalize
RUN set -ex; \
    chmod +x /usr/local/bin/sage; \
    chmod +x /usr/local/bin/demyx

EXPOSE 8080 3000

WORKDIR /var/www/html

USER demyx

ENTRYPOINT ["dumb-init", "demyx"]