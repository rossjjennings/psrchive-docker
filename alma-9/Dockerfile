FROM almalinux:9 AS build

# Install dependencies
RUN dnf install -y epel-release && dnf clean all # EPEL required for cfitsio, hdf5
RUN dnf config-manager --set-enabled crb # CRB required for swig
RUN dnf install -y --nogpgcheck \
    https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-9.noarch.rpm \
    https://mirrors.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-9.noarch.rpm \
    && dnf clean all # RPM Fusion required for pgplot

RUN dnf install -y \
    gcc-c++ \
    gcc-gfortran \
    autoconf \
    automake \
    libtool \
    libtool-ltdl \
    make \
    pkgconf-pkg-config \
    git \
    python3-devel \
    python3-pip \
    python3-numpy \
    fftw-devel \
    libX11-devel \
    libpng-devel \
    gsl-devel \
    cfitsio-devel \
    hdf5-devel \
    swig \
    pgplot \
    && dnf clean all

# PGPLOT
ENV PGPLOT_DIR="/usr/lib/pgplot5"
ENV PGPLOT_FONT="/usr/lib/pgplot5/grfont.dat"
ENV PGPLOT_INCLUDES="/usr/include"
ENV PGPLOT_BACKGROUND="white"
ENV PGPLOT_FOREGROUND="black"
ENV PGPLOT_DEV="/xs"

# TEMPO
WORKDIR /root
RUN git clone https://git.code.sf.net/p/tempo/tempo
WORKDIR /root/tempo
RUN autoreconf --install && \
    ./configure && \
    make && \
    make install
ENV TEMPO="/usr/local/share/tempo"
RUN mkdir $TEMPO && \
    cp -r clock $TEMPO/ && \
    cp -r ephem $TEMPO/ && \
    cp obsys.dat $TEMPO/ && \
    cp tempo.cfg $TEMPO/ && \
    cp tempo.hlp $TEMPO/

# Tempo2
WORKDIR /root
RUN git clone --depth=1 https://bitbucket.org/psrsoft/tempo2.git
WORKDIR /root/tempo2
RUN ./bootstrap && \
    ./configure --x-libraries=/usr/lib/x86_64-linux-gnu --enable-shared --enable-static F77=gfortran && \
    make -j $(nproc) && \
    make install && \
    make plugins-install
ENV TEMPO2="/usr/local/share/tempo2"
RUN cp -r T2runtime $TEMPO2

# PSRCHIVE
WORKDIR /root
RUN git clone --depth=1 https://git.code.sf.net/p/psrchive/code psrchive
WORKDIR /root/psrchive
RUN ./bootstrap && \
    ./configure --x-libraries=/usr/lib/x86_64-linux-gnu --enable-shared --enable-static F77=gfortran && \
    make -j $(nproc) && \
    make && \
    make install

FROM almalinux:9 AS deploy

# Install dependencies again (but only the runtime versions)
RUN dnf install -y epel-release && dnf clean all # EPEL required for cfitsio, hdf5
RUN dnf config-manager --set-enabled crb # CRB required for RPM Fusion
RUN dnf install -y --nogpgcheck \
    https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-9.noarch.rpm \
    https://mirrors.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-9.noarch.rpm \
    && dnf clean all # RPM Fusion required for pgplot

RUN dnf install -y \
    cfitsio \
    pgplot \
    python3-devel \
    python3-pip \
    python3-numpy \
    python3-scipy \
    python3-matplotlib \
    fftw-libs-single \
    libX11 \
    libpng \
    hdf5 \
    gsl \
    && dnf clean all
RUN alternatives --install /usr/bin/python python /usr/bin/python3 1

COPY --from=build /usr/local/ /usr/local/

ENV PGPLOT_DIR="/usr/lib/pgplot5"
ENV PGPLOT_FONT="/usr/lib/pgplot5/grfont.dat"
ENV PGPLOT_INCLUDES="/usr/include"
ENV PGPLOT_BACKGROUND="white"
ENV PGPLOT_FOREGROUND="black"
ENV PGPLOT_DEV="/xs"

ENV TEMPO="/usr/local/share/tempo"
ENV TEMPO2="/usr/local/share/tempo2"

ENV LD_LIBRARY_PATH="/usr/local/lib"
ENV PYTHONPATH="/usr/local/lib/python3.12/site-packages"
WORKDIR /root
