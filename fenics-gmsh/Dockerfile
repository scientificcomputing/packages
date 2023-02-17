FROM ghcr.io/scientificcomputing/fenics:2023-01-16-arm

# Dependency versions
ARG GMSH_VERSION=4_11_1
ARG OCC_VERSION=7_7_0

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

# FIXME: Need to make it conditional on the platform. For aarch64 the HDF5_DIR should be
# /usr/lib/aarch64-linux-gnu/ while on x86_64 it should be /usr/lib/x86_64-linux-gnu
ENV HDF5_MPI "ON" \
    HDF5_DIR "/usr/lib/aarch64-linux-gnu/"
RUN python3 -m pip install --no-cache-dir --no-binary=h5py git+https://github.com/h5py/h5py@master -vvv
RUN python3 -m pip install --no-cache-dir meshio

# GMSH installs python library in /usr/local/lib, see: https://gitlab.onelab.info/gmsh/gmsh/-/issues/1414
ENV PYTHONPATH=/usr/local/lib:$PYTHONPATH
ENV LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH

WORKDIR /root
RUN rm -rf /src