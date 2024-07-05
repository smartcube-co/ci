# Use the official .NET Core SDK image as the base image
FROM mcr.microsoft.com/dotnet/sdk:8.0-bookworm-slim

# Update base image
RUN echo 'deb http://deb.debian.org/debian bookworm-backports main' > /etc/apt/sources.list.d/backports.list \
    && apt-get update \
    && apt-get full-upgrade -y \
    && apt-get install --yes -t bookworm-backports git git-man zlib1g jq curl

# Install Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get update \
    && apt-get install --yes nodejs

# Install Angular, Yarn, Jest, ESLint, and TSLint
RUN npm install -g yarn jest eslint tslint sonarqube-scanner @angular/cli

# Install dotnet-coverage
RUN dotnet tool install --global dotnet-coverage

# Install SonarScanner for .NET
RUN dotnet tool install --global dotnet-sonarscanner

# Install Sonar Scanner dependencies
RUN apt-get update \
    && apt-get install --yes openjdk-17-jre-headless wget unzip gnupg git bash shellcheck locales

# Sonar Scanner CLI Environment Variables
ENV SONAR_SCANNER_HOME=/opt/sonar-scanner \
    JAVA_HOME=/usr/lib/jvm/java-17-openjdk \
    HOME=/tmp \
    XDG_CONFIG_HOME=/tmp \
    SONAR_USER_HOME=${SONAR_SCANNER_HOME}/.sonar \
    PATH=${SONAR_SCANNER_HOME}/bin:${PATH} \
    SRC_PATH=/usr/src \
    SCANNER_WORKDIR_PATH=/tmp/.scannerwork \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

# Install SonarScanner CLI
RUN SONAR_SCANNER_VERSION=$(curl -Ls https://api.github.com/repos/SonarSource/sonar-scanner-cli/releases/latest | jq .name -r) \
    && export SONAR_SCANNER_VERSION \
    && locale-gen en_US.UTF-8 \
    && wget -U "scannercli" -q -O /opt/sonar-scanner-cli.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SONAR_SCANNER_VERSION}.zip \
    && wget -U "scannercli" -q -O /opt/sonar-scanner-cli.zip.asc https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SONAR_SCANNER_VERSION}.zip.asc \
    && for server in $(shuf -e hkps://keys.openpgp.org hkps://keyserver.ubuntu.com) ; do \
        gpg --batch --keyserver "${server}" --recv-keys 679F1EE92B19609DE816FDE81DB198F93525EC1A && break || : ; \
    done \
    && gpg --verify /opt/sonar-scanner-cli.zip.asc /opt/sonar-scanner-cli.zip \
    && unzip /opt/sonar-scanner-cli.zip -d /opt \
    && mv /opt/sonar-scanner-${SONAR_SCANNER_VERSION} ${SONAR_SCANNER_HOME} \
    && rm /opt/sonar-scanner-cli.zip /opt/sonar-scanner-cli.zip.asc \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Make sure the tools are available in the PATH
ENV PATH="$PATH:/root/.dotnet/tools"

# Set the working directory
WORKDIR /app

# Default command (can be overridden in the CI/CD script)
CMD ["sh", "-c", "echo 'Image ready with .NET Core SDK, Angular CLI, SonarScanner for .NET, SonarScanner CLI, dotnet-coverage, Yarn, Jest, ESLint, and TSLint'"]
