#-----------------------------------------------------------------------------------------
# Basic Kubernetes Development Environmnet
#-----------------------------------------------------------------------------------------

FROM debian:buster as base

# Name of the base path
ARG BASE_PATH="/workspaces/angular-testing-course"
ENV PROJECT_PATH=$BASE_PATH


# Configure apt
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils 2>&1

# setup zsh and fonts
RUN apt-get -y install --no-install-recommends zsh fonts-powerline 2>&1

# Install git, process tools, lsb-release (common in install instructions for CLIs)
RUN apt-get -y install git procps lsb-release curl wget gnupg vim gawk unzip chromium
ENV CHROME_BIN='/usr/bin/chromium'

# courtesy entry to allow projects to stick utilities in the ./tools directory for easy execution
ENV PATH="${BASE_PATH}/tools:${PATH}"

# Install docker client (which will defer to your host docker)
RUN apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg |  apt-key add -
RUN  apt-key fingerprint 0EBFCD88
RUN  add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/debian \
    $(lsb_release -cs) \
    stable"
RUN  apt-get update \
    &&  apt-get install -y docker-ce-cli
# Install docker command line completion
RUN  curl -L https://raw.githubusercontent.com/docker/machine/v0.16.0/contrib/completion/bash/docker-machine.bash -o /etc/bash_completion.d/docker-machine


# Install jq
RUN curl -L https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 -o /usr/local/bin/jq &&  chmod +x /usr/local/bin/jq
# Install GCloud SDK
# RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && sudo apt-get update -y && sudo apt-get install google-cloud-sdk -y

# Install Azure tools
#RUN curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Install Chef Inspec: https://docs.chef.io/inspec/install/
RUN curl https://omnitruck.chef.io/install.sh |  bash -s -- -P inspec






# Setup Python
# RUN apt-get -y install python3-pip pylint
# RUN cp /usr/bin/pip3 /usr/bin/pip && chmod +x /usr/bin/pip



# Install X11 utilities so that we can run the Cypress UI.  Note, this requires an X11 server on your host!
# X11-apps is only here to help test.  It is not required and could be removed, no problem.
RUN  apt-get -y install x11-apps x11-xserver-utils
# Set display to go to the host X11 server.
ENV DISPLAY=host.docker.internal:0.0

# Install Node
ARG NODE_VERSION=14.17.0
RUN mkdir -p /usr/local/lib/nodejs && \
    curl -sL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz" -o "/tmp/node-v${NODE_VERSION}.tar.xz" && \
    tar -xvf "/tmp/node-v${NODE_VERSION}.tar.xz" -C /usr/local/lib/nodejs
ENV PATH="/usr/local/lib/nodejs/node-v${NODE_VERSION}-linux-x64/bin:${PATH}"

# Install dependencies for Cypress test tool
RUN apt-get update && apt-get install -y --no-install-recommends libgtk2.0-0 libgtk-3-0 libgbm-dev libnotify-dev libgconf-2-4 libnss3 libxss1 libasound2 libxtst6 xauth xvfb


# # Install Mochawesome
RUN npm install mochawesome mochawesome-merge mochawesome-report-generator --save-dev

# #
#  From cypress Dockerfile
#
RUN npm config -g set user $(whoami)
# point Cypress at the /root/cache no matter what user account is used
# see https://on.cypress.io/caching
# ENV CYPRESS_CACHE_FOLDER=/root/.cache/Cypress
RUN npm install -g "cypress@7.5.0"

RUN ls -la /root
RUN chmod 777 /root


# Install Oh-My-Zsh (for just this user)
RUN sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
ARG ZSH_CUSTOM="/root/.oh-my-zsh/custom"
RUN git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt"
RUN ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"

#
#  Configure spaceship and any other custom settings placed in config.sh (for just this user)
#
RUN sh -c "sed -i 's/robbyrussell/spaceship/g' /root/.zshrc"
COPY .devcontainer/config.sh /root
RUN cat /root/config.sh /root/.zshrc > /root/.zshrc_spaceship && cp /root/.zshrc_spaceship /root/.zshrc

# Install Firebase Angular
RUN env "PATH=${PATH}" npm install -g @angular/cli

# Install Firebase CLI
RUN env "PATH=${PATH}" npm install -g firebase-tools


#
# Install Salesforce CLI
#
# RUN curl -sL  "https://developer.salesforce.com/media/salesforce-cli/sfdx-cli/channels/stable/sfdx-cli-linux-x64.tar.xz" -o "/tmp/sfdx-cli-linux-x64.tar.xz"
# RUN mkdir /tmp/sfdx
# RUN tar xJf /tmp/sfdx-cli-linux-x64.tar.xz -C /tmp/sfdx --strip-components 1
# RUN /tmp/sfdx/install

# RUN mkdir /workspace
WORKDIR ${BASE_PATH}

# Clean up
RUN apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*
ENV DEBIAN_FRONTEND=dialog

FROM base as final

# Set the default shell to bash rather than sh
ENV SHELL /usr/bin/zsh
