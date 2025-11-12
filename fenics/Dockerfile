# Dependency versions
ARG PYBIND11_VERSION=2.10.1
ARG PETSC_VERSION=3.19.6
ARG SLEPC_VERSION=3.19.2
ARG UFL_BRANCH="main"
ARG DOLFIN_BRANCH="master"
ARG FFC_BRANCH="master"
ARG DIJITSO_BRANCH="master"
ARG FIAT_BRANCH="master"

# Flags to pass to ninja build system.
ARG MAKEFLAGS
# Compiler optimisation flags for SLEPc and PETSc, all languages.
# CMake build type for DOLFIN C++ build. See CMake documentation.
ARG DOLFIN_CMAKE_BUILD_TYPE="RelWithDebInfo"
# Extra CMake C++ compiler flags for DOLFIN C++ build.
ARG DOLFIN_CMAKE_CXX_FLAGS="-O2"

########################################

FROM ubuntu:24.04 AS fenics-base
# Usage of pip setuptools
ENV DEB_PYTHON_INSTALL_LAYOUT=deb_system

# PETSC vars
ENV PETSC_DIR=/usr/local/petsc
ENV PETSC_ARCH=linux-gnu-real-32
# SLEPC vars
ENV SLEPC_DIR=/usr/local/slepc

# Arguments that can be overridden by a user building the docker image
# Compiler optimisation flags for SLEPc and PETSc, all languages.
ARG PETSC_SLEPC_OPTFLAGS="-O2"
# Turn on PETSc and SLEPc debugging. "yes" or "no".
ARG PETSC_SLEPC_DEBUGGING="no"
# MPI variant. "mpich" or "openmpi".
ARG MPI="openmpi"
ARG PETSC_VERSION
ARG SLEPC_VERSION
ARG PYBIND11_VERSION

# Environment variables
ENV OPENBLAS_NUM_THREADS=1 \
    OPENBLAS_VERBOSE=0
ENV DEB_PYTHON_INSTALL_LAYOUT=deb_system
WORKDIR /tmp

# Install dependencies available via apt-get.
# - First set of packages are required to build and run FEniCS.
# - Second set of packages are recommended and/or required to build
#   documentation or tests.
RUN export DEBIAN_FRONTEND=noninteractive && \
    apt-get -qq update && \
    apt-get -yq --with-new-pkgs -o Dpkg::Options::="--force-confold" upgrade && \
    apt-get -y install \
    clang \
    cmake \
    g++ \
    git \
    gfortran \
    libboost-dev \
    libboost-filesystem-dev \
    libboost-iostreams-dev \
    libboost-math-dev \
    libboost-program-options-dev \
    libboost-system-dev \
    libboost-thread-dev \
    libboost-timer-dev \
    libeigen3-dev \
    libhdf5-${MPI}-dev \
    liblapack-dev \
    lib${MPI}-dev \
    libopenblas-dev \
    ninja-build \
    pkg-config \
    python3-dev \
    python3-pip \
    python3-setuptools \
    python3-numpy \
    python3-matplotlib \
    python3-scipy \
    python3-venv \
    wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV VIRTUAL_ENV=/fenics-env
ENV PATH=/fenics-env/bin:$PATH
RUN python3 -m venv ${VIRTUAL_ENV}

# Install Python packages (via pip)
RUN python3 -m pip install --upgrade pip "setuptools<=71.0.0"
RUN python3 -m pip install pytest pkgconfig
RUN python3 -m pip install --no-cache-dir mpi4py

# Install pybind11
RUN python3 -m pip install --no-cache-dir pybind11==${PYBIND11_VERSION} && \
    wget -nc --quiet https://github.com/pybind/pybind11/archive/v${PYBIND11_VERSION}.tar.gz && \
    tar -xf v${PYBIND11_VERSION}.tar.gz && \
    cd pybind11-${PYBIND11_VERSION} && \    
    mkdir build && \
    cd build && \
    cmake -DPYBIND11_TEST=False ../ && \
    make && \
    make install


# Install PETSc with real and complex types
WORKDIR ${PETSC_DIR}

RUN apt-get -qq update && \
    apt-get -y install bison flex && \
    wget -nc --quiet http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/petsc-lite-${PETSC_VERSION}.tar.gz -O petsc-${PETSC_VERSION}.tar.gz && \
    tar -xf petsc-${PETSC_VERSION}.tar.gz -C ${PETSC_DIR} --strip-components 1


# Real, 32-bit int
RUN python3 ./configure \
    --COPTFLAGS=${PETSC_SLEPC_OPTFLAGS} \
    --CXXOPTFLAGS=${PETSC_SLEPC_OPTFLAGS} \
    --FOPTFLAGS=${PETSC_SLEPC_OPTFLAGS} \
    --with-64-bit-indices=no \
    --with-debugging=${PETSC_SLEPC_DEBUGGING} \
    --with-fortran-bindings=no \
    --with-shared-libraries \
    --download-blacs \
    --download-hypre \
    --download-metis \
    --download-mumps \
    --download-ptscotch \
    --download-scalapack \
    --download-spai \
    --download-suitesparse \
    --download-superlu \
    --download-superlu_dist \
    --with-scalar-type=real && \
    make ${MAKEFLAGS} all


# Install petsc4py
WORKDIR ${PETSC_DIR}/src/binding/petsc4py
RUN python3 -m pip install "Cython==0.29.36" "numpy<2.0.0" --upgrade --no-cache-dir
RUN python3 -m pip install --no-cache-dir --no-build-isolation . -v

# Cleanup
WORKDIR /tmp  
RUN apt-get -y purge bison flex && \
    apt-get -y autoremove && \
    apt-get clean && \
    rm -rf \
    ${PETSC_DIR}/**/tests/ \
    ${PETSC_DIR}/**/obj/ \
    ${PETSC_DIR}/**/externalpackages/  \
    ${PETSC_DIR}/CTAGS \
    ${PETSC_DIR}/RDict.log \
    ${PETSC_DIR}/TAGS \
    ${PETSC_DIR}/docs/ \
    ${PETSC_DIR}/share/ \
    ${PETSC_DIR}/src/ \
    ${PETSC_DIR}/systems/ && \ 
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install SLEPc
RUN wget -nc --quiet https://slepc.upv.es/download/distrib/slepc-${SLEPC_VERSION}.tar.gz -O slepc-${SLEPC_VERSION}.tar.gz && \
    mkdir -p ${SLEPC_DIR} && tar -xf slepc-${SLEPC_VERSION}.tar.gz -C ${SLEPC_DIR} --strip-components 1 && \
    cd ${SLEPC_DIR} && \
    python3 ./configure && \
    make SLEPC_DIR=${SLEPC_DIR} && \
    # Install slepc4py
    cd src/binding/slepc4py && \
    python3 -m pip install --no-build-isolation --no-cache-dir . && \
    rm -rf ${SLEPC_DIR}/CTAGS ${SLEPC_DIR}/TAGS ${SLEPC_DIR}/docs ${SLEPC_DIR}/src/ ${SLEPC_DIR}/**/obj/ ${SLEPC_DIR}/**/test/ && \
    rm -rf /tmp/*


FROM fenics-base AS fenics-end-user
LABEL description="End user image"

ARG FIAT_BRANCH
ARG FFC_BRANCH
ARG UFL_BRANCH
ARG DOLFIN_BRANCH
ARG DIJITSO_BRANCH
ARG DOLFIN_CMAKE_BUILD_TYPE
ARG DOLFIN_CMAKE_CXX_FLAGS
ARG MAKEFLAGS

WORKDIR /src
ENV PETSC_DIR=/usr/local/petsc
ENV PETSC_ARCH=linux-gnu-real-32
ENV DEB_PYTHON_INSTALL_LAYOUT=deb_system

RUN git clone --branch ${FIAT_BRANCH} --single-branch  https://github.com/FEniCS/fiat.git && \
    git clone --branch ${FFC_BRANCH} --single-branch https://bitbucket.com/fenics-project/ffc.git && \
    git clone --branch ${UFL_BRANCH} --single-branch https://github.com/FEniCS/ufl-legacy.git && \
    git clone --branch ${DOLFIN_BRANCH} --single-branch https://bitbucket.com/fenics-project/dolfin.git && \
    git clone --branch ${DIJITSO_BRANCH} --single-branch https://bitbucket.com/fenics-project/dijitso.git

RUN cd fiat && python3 -m pip install --no-build-isolation --no-cache-dir . && \
    cd ../ufl-legacy && python3 -m pip install --no-build-isolation --no-cache-dir . && \
    cd ../dijitso && python3 -m pip install --no-build-isolation --no-cache-dir . && \
    cd ../ffc && python3 -m pip install --no-build-isolation --no-cache-dir . && \
    cd ../ && python3 -m pip install --no-build-isolation --no-cache-dir ipython
RUN python3 -m pip install --no-build-isolation "sympy<1.14"
RUN cd dolfin && \
    cmake -G Ninja -DCMAKE_BUILD_TYPE=${DOLFIN_CMAKE_BUILD_TYPE} -DCMAKE_CXX_FLAGS=${DOLFIN_CMAKE_CXX_FLAGS} -B build . && \
    ninja ${MAKEFLAGS} install -C build

RUN python3 -m pip install --no-dependencies --no-build-isolation ./dolfin/python

WORKDIR /root