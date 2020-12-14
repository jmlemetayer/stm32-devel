FROM	ubuntu:20.04 AS builder

COPY	defconfig /tmp/

RUN	set -x \
	# Install the needed packages
	&& export DEBIAN_FRONTEND=noninteractive \
	&& apt-get update \
	&& apt-get install --no-install-recommends -y \
		autoconf \
		automake \
		bison \
		build-essential \
		ca-certificates \
		file \
		flex \
		gawk \
		git \
		help2man \
		libncurses5-dev \
		libtool \
		libtool-bin \
		texinfo \
		unzip \
		wget \
	&& rm -rf /var/lib/apt/lists/* \
	# Install crosstool-ng
	&& git clone https://github.com/crosstool-ng/crosstool-ng /crosstool-ng \
	&& cd /crosstool-ng \
	&& ./bootstrap \
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
	&& echo "CROSS_COMPILE=\"$(ct-ng show-tuple)-\"" >> /etc/environment

FROM	ubuntu:20.04

COPY	--from=builder /x-tools/ /x-tools/
COPY	--from=builder /etc/environment /etc/environment
