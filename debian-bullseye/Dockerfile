FROM debian:bullseye

# Install dependencies (non-free needed for PGPLOT)
RUN sed -i "/^deb.*bullseye / s/$/ non-free/" /etc/apt/sources.list
RUN apt-get update -y && \
    apt-get --no-install-recommends -y install \
    build-essential \
    autoconf \
    autotools-dev \
    automake \
    autogen \
    libtool \
    pkg-config \
    csh \
    gcc \
    gfortran \
    wget \
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
    libxml2-dev \
    libltdl-dev \
    libgsl-dev \
    && rm -rf /var/lib/apt/lists/* \
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

ENV PSRHOME /software/
WORKDIR $PSRHOME 

# Pull all repos
RUN git clone https://bitbucket.org/psrsoft/tempo2.git && \
    git clone git://git.code.sf.net/p/psrchive/code psrchive

# tempo2
ENV TEMPO2 $PSRHOME/tempo2/T2runtime
ENV PATH $PATH:$PSRHOME/tempo2/T2runtime/bin
ENV C_INCLUDE_PATH $C_INCLUDE_PATH:$PSRHOME/tempo2/T2runtime/include
ENV LD_LIBRARY_PATH $LD_LIBRARY_PATH:$PSRHOME/tempo2/T2runtime/lib
WORKDIR $PSRHOME/tempo2
RUN sync && perl -pi -e 's/chmod \+x/#chmod +x/' bootstrap # Get rid of: returned a non-zero code: 126.
RUN ./bootstrap && \
    ./configure --x-libraries=/usr/lib/x86_64-linux-gnu --enable-shared --enable-static --with-pic F77=gfortran && \
    make -j $(nproc) && \
    make install && \
    make plugins-install && \
    rm -rf .git

# PSRCHIVE
ENV PSRCHIVE $PSRHOME/psrchive
ENV PATH $PATH:$PSRCHIVE/install/bin
ENV C_INCLUDE_PATH $C_INCLUDE_PATH:$PSRCHIVE/install/include
ENV LD_LIBRARY_PATH $LD_LIBRARY_PATH:$PSRCHIVE/install/lib
ENV PYTHONPATH $PSRCHIVE/install/lib/python3.9/site-packages
WORKDIR $PSRCHIVE
RUN ./bootstrap && \
    ./configure --prefix=$PSRCHIVE/install --x-libraries=/usr/lib/x86_64-linux-gnu --enable-shared --enable-static F77=gfortran && \
    make -j $(nproc) && \
    make && \
    make install && \
    rm -rf .git
WORKDIR $PSRHOME

WORKDIR /scripts/
