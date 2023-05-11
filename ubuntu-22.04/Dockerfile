FROM ubuntu:22.04 AS build

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
    libfftw3-dev \
    libx11-dev \
    libpng-dev \
    libhdf5-dev \
    libgsl-dev \
    && apt-get -y clean

# Install python packages
RUN pip install numpy scipy matplotlib

# PGPLOT
ENV PGPLOT_DIR /usr/lib/pgplot5
ENV PGPLOT_FONT /usr/lib/pgplot5/grfont.dat
ENV PGPLOT_INCLUDES /usr/include
ENV PGPLOT_BACKGROUND white
ENV PGPLOT_FOREGROUND black
ENV PGPLOT_DEV /xs

# tempo2
RUN git clone https://bitbucket.org/psrsoft/tempo2.git
WORKDIR tempo2
RUN ./bootstrap && \
    ./configure --x-libraries=/usr/lib/x86_64-linux-gnu --enable-shared --enable-static --with-pic F77=gfortran && \
    make -j $(nproc) && \
    make install && \
    make plugins-install
ENV TEMPO2 /usr/local/share/tempo2
RUN cp -r T2runtime $TEMPO2

# PSRCHIVE
RUN git clone git://git.code.sf.net/p/psrchive/code psrchive
WORKDIR psrchive
RUN ./bootstrap && \
    ./configure --x-libraries=/usr/lib/x86_64-linux-gnu --enable-shared --enable-static F77=gfortran && \
    make -j $(nproc) && \
    make && \
    make install

FROM ubuntu:22.04 AS deploy

# Install dependencies again (but only the runtime versions)
RUN apt-get update -y && apt-get install -y --no-install-recommends \
    libcfitsio9 \
    pgplot5 \
    python3-dev \
    libfftw3-single3 \
    libx11-6 \
    libpng16-16 \
    libhdf5-103-1 \
    libgsl27 \
    libgslcblas0

COPY --from=build /usr/local/ /usr/local/

ENV PGPLOT_DIR="/usr/lib/pgplot5"
ENV PGPLOT_FONT="/usr/lib/pgplot5/grfont.dat"
ENV PGPLOT_INCLUDES="/usr/include"
ENV PGPLOT_BACKGROUND="white"
ENV PGPLOT_FOREGROUND="black"
ENV PGPLOT_DEV="/xs"

ENV LD_LIBRARY_PATH="/usr/local/lib"
ENV PYTHONPATH="/usr/local/lib/python3.10/site-packages"
WORKDIR /root
