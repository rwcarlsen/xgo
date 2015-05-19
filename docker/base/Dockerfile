# Go cross compiler (xgo): Base cross-compilation layer
# Copyright (c) 2014 Péter Szilágyi. All rights reserved.
#
# Released under the MIT license.

FROM ubuntu:14.04

MAINTAINER Péter Szilágyi <peterke@gmail.com>

# Configure the Go environment, since it's not going to change
ENV PATH   /usr/local/go/bin:$PATH
ENV GOPATH /go


# Inject the remote file fetcher and checksum verifier
ADD fetch.sh /fetch.sh
ENV FETCH /fetch.sh
RUN chmod +x $FETCH


# Make sure apt-get is up to date and dependent packages are installed
RUN \
  apt-get update && \
  apt-get install -y automake autogen build-essential ca-certificates \
    gcc-arm-linux-gnueabi libc6-dev-armel-cross gcc-multilib gcc-mingw-w64 \
    clang llvm-dev  libtool libxml2-dev uuid-dev libssl-dev pkg-config \
    patch make xz-utils cpio wget unzip git mercurial --no-install-recommends

RUN apt-get install -y g++-multilib --no-install-recommends
RUN apt-get install -y g++-mingw-w64 --no-install-recommends

# Configure the container for OSX cross compilation
ENV OSX_SDK_PATH https://github.com/trevd/android_platform_build2/raw/master/osxsdks10.6.tar.gz
ENV OSX_SDK      MacOSX10.6.sdk

RUN \
  git clone https://github.com/tpoechtrager/osxcross.git && \
  sed -i.bak s/read/#read/g /osxcross/build.sh && \
  \
  $FETCH $OSX_SDK_PATH f526b4ae9806e8d31e3b094e3f004f8f160a3fad && \
  tar -xzf `basename $OSX_SDK_PATH` --strip-components 1 SDKs/$OSX_SDK && \
  tar -cjf /osxcross/tarballs/$OSX_SDK.tar.bz2 $OSX_SDK && \
  rm -rf `basename $OSX_SDK_PATH` $OSX_SDK && \
  \
  /osxcross/build.sh
ENV PATH /osxcross/target/bin:$PATH


# Inject the Go package downloader and tool-chain bootstrapper
ADD bootstrap.sh /bootstrap.sh
ENV BOOTSTRAP /bootstrap.sh
RUN chmod +x $BOOTSTRAP

# Inject the C dependency cross compiler
ADD build_deps.sh /build_deps.sh
ENV BUILD_DEPS /build_deps.sh
RUN chmod +x $BUILD_DEPS

# Inject the container entry point, the build script
ADD build.sh /build.sh
ENV BUILD /build.sh
RUN chmod +x $BUILD

# Configure the Go packages and bootstrap them
RUN \
  export DIST_LINUX_64=https://storage.googleapis.com/golang/go1.4.2.linux-amd64.tar.gz && \
  export DIST_LINUX_64_SHA1=5020af94b52b65cc9b6f11d50a67e4bae07b0aff && \
  \
  export DIST_LINUX_32=https://storage.googleapis.com/golang/go1.4.2.linux-386.tar.gz && \
  export DIST_LINUX_32_SHA1=50557248e89b6e38d395fda93b2f96b2b860a26a && \
  \
  export DIST_LINUX_ARM=http://dave.cheney.net/paste/go1.4.2.linux-arm~armv5-1.tar.gz && \
  export DIST_LINUX_ARM_SHA1=1bcfc8ef9c2aa381722b71b8c8d83cb58e973116 && \
  \
  export DIST_OSX_64=https://storage.googleapis.com/golang/go1.4.2.darwin-amd64-osx10.6.tar.gz && \
  export DIST_OSX_64_SHA1=00c3f9a03daff818b2132ac31d57f054925c60e7 && \
  \
  export DIST_OSX_32=https://storage.googleapis.com/golang/go1.4.2.darwin-386-osx10.6.tar.gz && \
  export DIST_OSX_32_SHA1=fb3e6b30f4e1b1be47bbb98d79dd53da8dec24ec && \
  \
  export DIST_WIN_64=https://storage.googleapis.com/golang/go1.4.2.windows-amd64.zip && \
  export DIST_WIN_64_SHA1=91b229a3ff0a1ce6e791c832b0b4670bfc5457b5 && \
  \
  export DIST_WIN_32=https://storage.googleapis.com/golang/go1.4.2.windows-386.zip && \
  export DIST_WIN_32_SHA1=0e074e66a7816561d7947ff5c3514be96f347dc4 && \
  \
  $BOOTSTRAP

ENTRYPOINT ["/build.sh"]
