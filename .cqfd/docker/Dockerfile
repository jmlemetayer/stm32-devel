FROM	ubuntu:20.04 AS builder

ARG	CROSSTOOL_NG_DIR=crosstool-ng-1.24.0
ARG	CROSSTOOL_NG_TGZ=${CROSSTOOL_NG_DIR}.tar.bz2
ARG	CROSSTOOL_NG_URL=http://crosstool-ng.org/download/crosstool-ng/${CROSSTOOL_NG_TGZ}
ARG	CROSSTOOL_NG_MD5=a373aebacc7ad54655506b1fbe5dbe63

COPY	defconfig /tmp/

RUN	set -x \
	# Install the needed packages
	&& export DEBIAN_FRONTEND=noninteractive \
	&& apt-get update \
	&& apt-get install --no-install-recommends -y \
		bison \
		build-essential \
		ca-certificates \
		file \
		flex \
		gawk \
		help2man \
		libncurses5-dev \
		libtool-bin \
		texinfo \
		unzip \
		wget \
	&& rm -rf /var/lib/apt/lists/* \
	# Download crosstool-ng
	&& wget ${CROSSTOOL_NG_URL} \
	&& echo "${CROSSTOOL_NG_MD5}  ${CROSSTOOL_NG_TGZ}" | md5sum -c - \
	&& tar -C / -xjf ${CROSSTOOL_NG_TGZ} \
	&& rm -f ${CROSSTOOL_NG_TGZ} \
	# Install crosstool-ng
	&& cd /${CROSSTOOL_NG_DIR} \
	&& ./configure --prefix=/usr \
	&& make \
	&& make install \
	# Build the toolchain
	&& mkdir /build \
	&& cd /build \
	&& cp /tmp/defconfig . \
	&& ct-ng defconfig \
	&& sed -i 's|^\(# \)*\(CT_PREFIX_DIR\)[= ].*|\2="/x-tools"|' .config \
	&& ct-ng CT_ALLOW_BUILD_AS_ROOT_SURE=y build \
	# Configure the environment
	&& sed -i 's|^\(PATH="\)\(.*\)|\1/x-tools/bin:\2|' /etc/environment \
	&& echo "CROSS_COMPILE=\"$(ct-ng show-tuple)-\"" >> /etc/environment \
	# Clean the image
	&& rm -rf /crosstool-ng-1.24.0 /build

FROM	ubuntu:20.04

# Copy the toolchain and environment from the builder
COPY	--from=builder /x-tools/ /x-tools/
COPY	--from=builder /etc/environment /etc/environment

RUN	set -x \
	# Install the needed packages
	&& export DEBIAN_FRONTEND=noninteractive \
	&& apt-get update \
	&& apt-get install --no-install-recommends -y \
		make \
		python3 \
		stlink-tools \
	&& rm -rf /var/lib/apt/lists/*
