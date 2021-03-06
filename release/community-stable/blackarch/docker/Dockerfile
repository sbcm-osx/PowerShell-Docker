# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License.
#
# Docker image file that describes an BlackArch Linux image (derived from Arch
# Linux base image) with PowerShell installed from full PowerShell linux tar.gz
# package

# Define arg(s) needed for the From statement
ARG fromTag=2018.10.01
ARG imageRepo=base/archlinux

FROM ${imageRepo}:${fromTag} AS installer-env

# Define Args for the needed for BlackArch Linux
ARG BlackArch_Strap_URL=https://blackarch.org/strap.sh

# Download BlackArch Linux strap shell script
ADD ${BlackArch_Strap_URL} /tmp/strap.sh

# Define Args for the needed to add the package
ARG PS_VERSION=6.1.0
ARG PS_PACKAGE=powershell-${PS_VERSION}-linux-x64.tar.gz
ARG PS_PACKAGE_URL=https://github.com/PowerShell/PowerShell/releases/download/v${PS_VERSION}/${PS_PACKAGE}
ARG PS_INSTALL_VERSION=6

# Download the Linux tar.gz and save it
ADD ${PS_PACKAGE_URL} /tmp/powershell-linux.tar.gz

# Define Args and Env needed to create links
ENV PS_INSTALL_FOLDER=/opt/microsoft/powershell/$PS_INSTALL_VERSION \
    \
    # Define ENVs for Localization/Globalization
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    # Opt out of SocketsHttpHandler in DotNet Core 2.1 to use HttpClientHandler
    # with installed libcurl4 package to resolve 
    # Invoke-WebRequest : Authentication failed" issue when executing using
    # docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [Invoke-WebRequest] [-Uri <HTTPS URL>]
    DOTNET_SYSTEM_NET_HTTP_USESOCKETSHTTPHANDLER=0

# Installation
RUN \
    # enable en_US.UTF-8 locale
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
    # generate locale
    && locale-gen \
    # set execute permission on BlackArch strap shell script
    && chmod +x /tmp/strap.sh \
    # run BlackArch strap shell script
    && /tmp/strap.sh \
    # remove BlackArch strap shell script
    && rm -f /tmp/strap.sh \
    # update package database
    && pacman -Syy \
    # install dependencies
    && pacman -S --noconfirm \
      # required for International Components for Unicode
      core/icu \
      # required for SSL
      openssl-1.0 \
    # create powershell folder
    && mkdir -p ${PS_INSTALL_FOLDER} \
    # uncompress powershell linux tar file
    && tar zxf /tmp/powershell-linux.tar.gz -C ${PS_INSTALL_FOLDER} \
    # remove powershell linux tar file
    && rm -f /tmp/powershell-linux.tar.gz \
    # Create the pwsh symbolic link that points to powershell
    && ln -s ${PS_INSTALL_FOLDER}/pwsh /usr/bin/pwsh \
    # upgrade distro
    && pacman -Syyu --noconfirm \
    # clean downloaded packages
    && yes | pacman -Scc

# Define args needed only for the labels
ARG IMAGE_NAME=pshorg/powershellcommunity:blackarch
ARG VCS_REF="none"

# Add label last as it's just metadata and uses a lot of parameters
LABEL maintainer="PowerShell Team <powershellteam@hotmail.com>" \
      readme.md="https://github.com/PowerShell/PowerShell/blob/master/docker/README.md" \
      description="This Dockerfile will install the latest release of PowerShell." \
      org.label-schema.usage="https://github.com/PowerShell/PowerShell/tree/master/docker#run-the-docker-image-you-built" \
      org.label-schema.url="https://github.com/PowerShell/PowerShell/blob/master/docker/README.md" \
      org.label-schema.vcs-url="https://github.com/PowerShell/PowerShell-Docker" \
      org.label-schema.name="powershell" \
      org.label-schema.vendor="PowerShell" \
      org.label-schema.vcs-ref=${VCS_REF} \
      org.label-schema.version=${PS_VERSION} \
      org.label-schema.schema-version="1.0" \
      org.label-schema.docker.cmd="docker run ${IMAGE_NAME} pwsh -c '$psversiontable'" \
      org.label-schema.docker.cmd.devel="docker run ${IMAGE_NAME}" \
      org.label-schema.docker.cmd.test="docker run ${IMAGE_NAME} pwsh -c Invoke-Pester" \
      org.label-schema.docker.cmd.help="docker run ${IMAGE_NAME} pwsh -c Get-Help"

CMD [ "pwsh" ]
