FROM ubuntu:18.04

SHELL ["/bin/bash", "-c"]

ADD "https://www.sofa-framework.org/rand/" ForceNoCache

# Install APT tools and repos
RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y \
    apt-utils \
    software-properties-common
RUN add-apt-repository ppa:git-core/ppa
RUN apt-get update && apt-get upgrade -y

# Install system tools
RUN apt-get install -y \
    git \
    net-tools \
    vim \
    wget \
    curl \
    zip \
    unzip \
    patchelf

# Install compilers
RUN apt-get install -y \
    build-essential \
    ninja-build \
    gcc-7 g++-7 \
    clang \
    ccache

# Install core deps
RUN apt-get install -y \
    libglew-dev \
    freeglut3-dev \
    zlib1g-dev \
    libeigen3-dev

# Install plugins deps
RUN apt-get install -y \
    python2.7-dev python-pip \
    python3.7-dev python3-pip \
    libpng-dev libjpeg-dev libtiff-dev \
    libblas-dev \
    liblapack-dev \
    libsuitesparse-dev \
    libavcodec-dev libavformat-dev libavutil-dev libswscale-dev \
    libassimp-dev \
    libbullet-dev \
    liboce-ocaf-dev \
    libzmq3-dev liboscpack-dev \
    libxrandr-dev libxinerama-dev libxcursor-dev libxi-dev
RUN python2.7 -m pip install --upgrade "pip == 20.3.4" \
    && python2.7 -m pip install "numpy == 1.16.6" "scipy == 1.2.3" "matplotlib == 2.2.5"
RUN python3.7 -m pip install --upgrade pip \
    && python3.7 -m pip install numpy scipy pygame
ENV VM_HAS_ASSIMP="true"
# BulletColisionDetection is broken
ENV VM_HAS_BULLET="disabled"
ENV VM_HAS_OPENCASCADE="true"

###################################

# Install CMake
ARG CMAKE_MAJOR=3
ARG CMAKE_MINOR=12
ARG CMAKE_PATCH=0
ADD https://github.com/Kitware/CMake/releases/download/v$CMAKE_MAJOR.$CMAKE_MINOR.$CMAKE_PATCH/cmake-$CMAKE_MAJOR.$CMAKE_MINOR.$CMAKE_PATCH-Linux-x86_64.sh /tmp/cmake.sh
RUN chmod a+x /tmp/cmake.sh
RUN /tmp/cmake.sh --skip-license --prefix=/usr/local

# Install Qt
ARG QT_MAJOR=5
ARG QT_MINOR=12
ARG QT_PATCH=6
RUN add-apt-repository -y ppa:beineri/opt-qt-$QT_MAJOR.$QT_MINOR.$QT_PATCH-bionic \
    && apt-get update \
    && apt-get install -y qt${QT_MAJOR}${QT_MINOR}-meta-full qt${QT_MAJOR}${QT_MINOR}charts-no-lgpl \
    && rm -rf /opt/qt${QT_MAJOR}${QT_MINOR}/doc \
    && rm -rf /opt/qt${QT_MAJOR}${QT_MINOR}/examples
ADD https://www.sofa-framework.org/wp-content/uploads/2020/06/QtInstallerFramework_3.0_Linux.zip /tmp
RUN unzip /tmp/QtInstallerFramework_3.0_Linux.zip -d /builds && chmod -R a+x /builds/Qt
ENV QTIFWDIR="/builds/Qt/Tools/QtInstallerFramework/3.0"
ENV PATH="${QTIFWDIR}/bin:${PATH}"

# Install Boost
ARG BOOST_MAJOR=1
ARG BOOST_MINOR=67
RUN add-apt-repository -y ppa:mhier/libboost-latest \
    && apt-get update \
    && apt-get install -y libboost$BOOST_MAJOR.$BOOST_MINOR-dev

# Install CGAL
# Due to dependencies on Boost and Qt, we have to build CGAL
ARG CGAL_MAJOR=4
ARG CGAL_MINOR=14
ARG CGAL_PATCH=3
ADD https://github.com/CGAL/cgal/releases/download/releases/CGAL-$CGAL_MAJOR.$CGAL_MINOR.$CGAL_PATCH/CGAL-$CGAL_MAJOR.$CGAL_MINOR.$CGAL_PATCH.tar.xz /tmp/CGAL.tar.xz
RUN apt-get install -y libgmp-dev libmpfr-dev
RUN tar -xJf /tmp/CGAL.tar.xz --directory /tmp \
    && cd /tmp/CGAL-$CGAL_MAJOR.$CGAL_MINOR.$CGAL_PATCH \
    && mkdir build \
    && cd build \
    && cmake -DCMAKE_BUILD_TYPE=Release -DWITH_CGAL_Core=TRUE -DWITH_CGAL_ImageIO=TRUE -DWITH_CGAL_Qt5=TRUE .. \
    && make --silent install
ENV VM_HAS_CGAL="true"
ENV VM_CGAL_PATH="/usr/local/lib/cmake/CGAL"

# Install CUDA
RUN apt-get install -y nvidia-cuda-toolkit
ENV VM_HAS_CUDA="true"
ENV VM_CUDA_HOST_COMPILER="/usr/bin/gcc-6"
ENV VM_CUDA_ARCH="sm_50"

# Install pybind11 (needed by SofaPython3)
ARG PYBIND_MAJOR=2
ARG PYBIND_MINOR=6
RUN git clone -b v$PYBIND_MAJOR.$PYBIND_MINOR --depth 1 https://github.com/pybind/pybind11.git /tmp/pybind11/src \
    && mkdir /tmp/pybind11/build && cd /tmp/pybind11/build \
    && cmake -DCMAKE_BUILD_TYPE=Release -DPYTHON_EXECUTABLE=/usr/bin/python3.7 -DPYBIND11_TEST=OFF ../src \
    && make --silent install

# Install NodeEditor (needed by SofaQtQuick)
ARG NODE_MAJOR=2
ARG NODE_MINOR=1
ARG NODE_PATCH=3
RUN git clone -b $NODE_MAJOR.$NODE_MINOR.$NODE_PATCH --depth 1 https://github.com/paceholder/nodeeditor.git /tmp/nodeeditor/src \
    && mkdir /tmp/nodeeditor/build && cd /tmp/nodeeditor/build \
    && cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_EXAMPLES=OFF -DBUILD_TESTING=OFF ../src \
    && make --silent install

###################################

# Disable core dumps
RUN printf '\n# Disable core dumps\n* soft core 0\n* hard core 0\n' >> /etc/security/limits.conf

# Init /builds directory
WORKDIR /builds

# Set env vars
ENV HOME="/root"
ENV PYTHONIOENCODING="UTF-8"
ENV PYTHONUSERBASE="/tmp/pythonuserbase"
RUN mkdir -p "$PYTHONUSERBASE" && chmod -R 777 "$PYTHONUSERBASE"
ENV PYTHON3_EXECUTABLE="/usr/bin/python3.7"

# Cleanup
RUN apt-get clean -y \
    && apt-get autoremove -y \
    && rm -rf /tmp/*

# In-process env settings
COPY docker-entrypoint.sh /
RUN chmod a+x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

# Define default command.
CMD ["/bin/bash"]
