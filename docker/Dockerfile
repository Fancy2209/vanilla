FROM ubuntu:20.04

# Update apt
RUN apt -y update

# Install latest CMake
RUN DEBIAN_FRONTEND=noninteractive TZ=Etc/UTC \
        apt -y install build-essential ninja-build libssl-dev git tzdata
RUN git clone --depth 1 --branch v3.29.3 https://gitlab.kitware.com/cmake/cmake.git \
        && cd cmake \
        && mkdir build \
        && cd build \
        && ../configure \
        && make install \
        && cd ../.. \
        && rm -rf cmake

# Install appimagetool
RUN apt -y install pkg-config libgpgme-dev libgcrypt-dev \
        libglib2.0-dev libcurl-ocaml-dev squashfs-tools \
        desktop-file-utils
RUN git clone https://github.com/AppImage/appimagetool.git \
        && cd appimagetool \
        && mkdir build \
        && cd build \
        && cmake .. -G Ninja \
        && ninja install \
        && cd ../.. \
        && rm -rf appimagetool

# Get linuxdeploy
RUN apt -y install patchelf libturbojpeg0-dev wget cimg-dev libpng-dev
RUN git clone --recursive https://github.com/linuxdeploy/linuxdeploy.git \
        && cd linuxdeploy \
        && mkdir build \
        && cd build \
        && cmake .. -G Ninja \
        && ninja install \
        && cd ../.. \
        && rm -rf linuxdeploy

# Get linuxdeploy-plugin-qt
RUN apt -y install nlohmann-json3-dev
RUN git clone --recursive https://github.com/linuxdeploy/linuxdeploy-plugin-qt.git \
        && cd linuxdeploy-plugin-qt \
        && mkdir build \
        && cd build \
        && cmake .. -G Ninja \
        && ninja install \
        && cd ../.. \
        && rm -rf linuxdeploy-plugin-qt

# Build Qt 6
RUN apt -y install libgles2-mesa-dev libegl1-mesa-dev libfontconfig-dev \
        libxkbcommon-x11-dev libx11-xcb-dev libpulse-dev \
        libxcb-glx0-dev libxcb-cursor-dev libxcb-icccm4-dev \
        libxcb-image0-dev libxcb-keysyms1-dev libxcb-randr0-dev \
        libxcb-render0-dev libxcb-render-util0-dev libxcb-shape0-dev \
        libxcb-shm0-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-xkb-dev
RUN git clone --branch 6.7.1 --depth 1 git://code.qt.io/qt/qt5.git qt6 \
    && cd qt6 \
    && perl init-repository --module-subset=qtbase,qtmultimedia,qtshadertools \
    && mkdir qt6-build \
    && cd qt6-build \
    && ../configure -xcb -verbose -no-feature-ffmpeg -prefix /usr/local \
    && cmake --build . --parallel \
    && cmake --install . \
    && cd .. \
    && rm -rf qt6

# Other dependencies for Vanilla/AppImages
RUN apt -y install libavcodec-dev libavfilter-dev libavutil-dev \
        libsdl2-dev libnl-genl-3-dev \
        isc-dhcp-client

# Build Vanilla and make an AppImage
CMD git config --global --add safe.directory /vanilla/lib/hostap \
        && cp /vanilla/lib/hostap/conf/wpa_supplicant.config /vanilla/lib/hostap/wpa_supplicant/.config \
        && /usr/local/bin/cmake /vanilla -B/build -G Ninja -DCMAKE_INSTALL_PREFIX=/AppDir/usr \
        && cmake --build /build \
        && cmake --install /build \
        && cp $(which dhclient) /AppDir/usr/bin \
        && cp /usr/lib/$(uname -m)-linux-gnu/libOpenGL.so* /AppDir/usr/lib \ # Ubuntu complains about this missing but supposedly it's a system lib we shouldn't be including?
        && linuxdeploy --appdir=/AppDir --plugin qt \
        && appimagetool /AppDir \
        && mv Vanilla*.AppImage /vanilla
