ARG IRODS_VERSION="4.2.11"
ARG HTSLIB_VERSION=1.22.1
ARG SAMTOOLS_VERSION=1.22.1
ARG BCFTOOLS_VERSION=1.22.1
ARG DEFLATE_VERSION=1.24

FROM ubuntu:18.04 AS samtools_build

ARG IRODS_VERSION
ARG HTSLIB_VERSION
ARG SAMTOOLS_VERSION
ARG BCFTOOLS_VERSION
ARG DEFLATE_VERSION

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get upgrade -y && \
    apt-get install -qq -y --no-install-recommends \
      git \
      ca-certificates \
      autotools-dev \
      build-essential \
      autoconf \
      automake \
      libbz2-dev \
      libcurl4-openssl-dev \
      libgd-dev \
      liblzma-dev \
      libssl-dev \
      libncurses5-dev \
      curl \
      dirmngr \
      git \
      gpg \
      gpg-agent \
      cmake \
      lsb-release \
      locales && \
      locale-gen en_GB en_GB.UTF-8 && \
      localedef -i en_GB -c -f UTF-8 -A /usr/share/locale/locale.alias en_GB.UTF-8

ENV LANG=en_GB.UTF-8 \
    LANGUAGE=en_GB \
    LC_ALL=en_GB.UTF-8

RUN curl -sSL https://packages.irods.org/irods-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" |\
    tee /etc/apt/sources.list.d/renci-irods.list && \
    apt-get update && \
    apt-get install -q -y --no-install-recommends \
    irods-dev="${IRODS_VERSION}-1~$(lsb_release -sc)" \
    irods-runtime="${IRODS_VERSION}-1~$(lsb_release -sc)"

RUN cd /tmp && \
    curl -sSL -O https://github.com/ebiggers/libdeflate/releases/download/v${DEFLATE_VERSION}/libdeflate-${DEFLATE_VERSION}.tar.gz && \
    tar xzf libdeflate-${DEFLATE_VERSION}.tar.gz && \
    cd libdeflate-${DEFLATE_VERSION} && \
    cmake . && \
    make install

RUN cd /tmp && \
    curl -sSL -O "https://github.com/samtools/htslib/releases/download/${HTSLIB_VERSION}/htslib-${HTSLIB_VERSION}.tar.bz2" && \
    tar xfj htslib-${HTSLIB_VERSION}.tar.bz2 && \
    cd htslib-${HTSLIB_VERSION} && \
    ./configure --enable-plugins --with-libdeflate && \
    make && \
    make install && \
    ldconfig

RUN cd /tmp && \
    curl -sSL -O "https://github.com/samtools/htslib-plugins/archive/refs/tags/201712.tar.gz" && \
    tar xfz 201712.tar.gz && \
    cd htslib-plugins-201712 && \
    export IRODS_HOME=/usr/include/irods && \
    make && \
    make install

RUN cd /tmp && \
    curl -sSL -O "https://github.com/samtools/samtools/releases/download/${SAMTOOLS_VERSION}/samtools-${SAMTOOLS_VERSION}.tar.bz2" && \
    tar xfj samtools-${SAMTOOLS_VERSION}.tar.bz2 && \
    cd samtools-${SAMTOOLS_VERSION} && \
    ./configure --with-htslib=/tmp/htslib-${HTSLIB_VERSION} && \
    make install

RUN cd /tmp && \
    curl -sSL -O "https://github.com/samtools/bcftools/releases/download/${BCFTOOLS_VERSION}/bcftools-${BCFTOOLS_VERSION}.tar.bz2" && \
    tar xfj bcftools-${BCFTOOLS_VERSION}.tar.bz2 && \
    cd bcftools-${BCFTOOLS_VERSION} && \
    ./configure --with-htslib=/tmp/htslib-${HTSLIB_VERSION} && \
    make install

#
# Next stage: make everything smaller
#
FROM ubuntu:18.04

ARG IRODS_VERSION
ARG DEFLATE_VERSION

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get upgrade -y && \
    apt-get install -qq -y --no-install-recommends \
      ca-certificates \
      curl \
      libcurl4 \
      libssl-dev \
      lsb-release \
      gnuplot \
      gpg \
      gpg-agent \
      perl \
      locales && \
    locale-gen en_GB en_GB.UTF-8 && \
    localedef -i en_GB -c -f UTF-8 -A /usr/share/locale/locale.alias en_GB.UTF-8

RUN curl -sSL https://packages.irods.org/irods-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" |\
    tee /etc/apt/sources.list.d/renci-irods.list && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      irods-runtime="${IRODS_VERSION}-1~$(lsb_release -sc)"

RUN apt-get autoremove --yes && \
    apt-get clean --yes

COPY --from=samtools_build /usr/local /usr/local

# Copy the singularity-wrapper scripts
ARG DOCKER_IMAGE
ARG DOCKER_TAG
ENV DOCKER_IMAGE="$DOCKER_IMAGE"
ENV DOCKER_TAG="$DOCKER_TAG"

RUN ldconfig

COPY --from=singularity ./scripts/* /usr/local/bin/
COPY ./manifest.txt /opt/wtsi-npg/etc/manifest.txt

# Create and switch to New user
ARG USER=appuser
ARG UID=1000
ARG GID=$UID

RUN groupadd --gid $GID $USER && \
    useradd --uid $UID --gid $GID --shell /bin/bash --create-home $USER

USER $USER

CMD ["/bin/bash"]

