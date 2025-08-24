FROM ubuntu:24.04 AS build

# Install dependencies
RUN apt-get update -y && apt-get install -y --no-install-recommends \
    g++ \
    gfortran \
    autoconf \
    automake \
    libtool \
    libltdl-dev \
    make \
    pkg-config \
    git \
    libcfitsio-dev \
    pgplot5 \
    swig \
    python3-dev \
    python3-pip \
    python3-numpy \
    libfftw3-dev \
    libx11-dev \
    libpng-dev \
    libhdf5-dev \
    libgsl-dev \
    && apt-get clean -y

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

FROM ubuntu:24.04 AS deploy

# Install dependencies again (but only the runtime versions)
RUN apt-get update -y && apt-get install -y --no-install-recommends \
    libcfitsio10 \
    pgplot5 \
    python3-dev \
    python3-pip \
    python3-numpy \
    python3-scipy \
    python3-matplotlib \
    python3-astropy \
    python-is-python3 \
    libfftw3-single3 \
    libx11-6 \
    libpng16-16t64 \
    libhdf5-103-1t64 \
    libgsl27 \
    libgslcblas0 \
    && apt-get clean -y

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
