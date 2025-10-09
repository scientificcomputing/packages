FROM ghcr.io/scientificcomputing/fenics:2025-10-09

# Dependency versions
ARG GMSH_VERSION=4_14_1

WORKDIR /tmp

# Install dependencies available via apt-get.
RUN export DEBIAN_FRONTEND=noninteractive && \
    apt-get -qq update && \
    apt-get -yq --with-new-pkgs -o Dpkg::Options::="--force-confold" upgrade && \
    apt-get -y install \
    libglu1 \
    libxcursor-dev \
    libxft2 \
    libxinerama1 \
    libfltk1.3-dev \
    libfreetype6-dev  \
    libgl1-mesa-dev \
    libocct-foundation-dev \
    libocct-data-exchange-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


# Install OpenCASACADE
# OpenCASACADE is currently installed via apt, but the following lines
# would be way to install this from source in case this is needed later
# ARG OCC_VERSION=7_7_0
# RUN curl -L -o occt.tgz "http://git.dev.opencascade.org/gitweb/?p=occt.git;a=snapshot;h=refs/tags/V${OCC_VERSION};sf=tgz" && \
#     tar zxf occt.tgz && \
#     cmake -G Ninja -DBUILD_MODULE_Draw:BOOL=FALSE -DCMAKE_BUILD_TYPE=Release -B build-occt -S occt-V${OCC_VERSION} && \
#     cmake --build build-occt && \
#     cmake --install build-occt && \
#     rm -rf /tmp/*

# Install gmsh
RUN git clone -b gmsh_${GMSH_VERSION} --single-branch --depth 1 https://gitlab.onelab.info/gmsh/gmsh.git && \
    cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DENABLE_BUILD_DYNAMIC=1 -B build-dir -S gmsh && \
    cmake --build build-dir && \
    cmake --install build-dir && \
    rm -rf /tmp/*

# GMSH installs python package in /usr/local/lib, see: https://gitlab.onelab.info/gmsh/gmsh/-/issues/1414
RUN export SP_DIR=$(python3 -c 'import sys, site; sys.stdout.write(site.getsitepackages()[0])') \
    && mv /usr/local/lib/gmsh.py ${SP_DIR}/ \
    && mv /usr/local/lib/gmsh*.dist-info ${SP_DIR}/

# build h5py from source
# disable build isolation due to https://github.com/h5py/h5py/issues/2222
RUN export HDF5_MPI=ON \
    && export HDF5_PKGCONFIG_NAME="hdf5" \
    && python3 -m pip install cython pkgconfig mpi4py \
    && python3 -m pip install --no-cache-dir --no-binary=h5py --no-build-isolation h5py -vv
RUN python3 -m pip install --no-cache-dir meshio

WORKDIR /
