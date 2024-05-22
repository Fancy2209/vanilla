FROM ubuntu:20.04

# Update apt
RUN apt -y update

# Dependencies for Qt 6
RUN DEBIAN_FRONTEND=noninteractive TZ=Etc/UTC \
        apt -y install build-essential ninja-build cmake git \
                libgles2-mesa-dev libegl1-mesa-dev libfontconfig-dev \
                libxkbcommon-x11-dev libx11-xcb-dev  libpulse-dev \
                libxcb-glx0-dev libxcb-cursor-dev libxcb-icccm4-dev \
                libxcb-image0-dev libxcb-keysyms1-dev libxcb-randr0-dev \
                libxcb-render0-dev libxcb-render-util0-dev libxcb-shape0-dev \
                libxcb-shm0-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-xkb-dev

# Build Qt 6
RUN git clone --branch 6.7.1 --depth 1 git://code.qt.io/qt/qt5.git qt6 \
    && cd qt6 \
    && perl init-repository --module-subset=qtbase,qtmultimedia,qtshadertools \
    && mkdir qt6-build \
    && cd qt6-build \
    && ../configure -xcb -verbose -prefix /usr/local \
    && cmake --build . --parallel \
    && cmake --install . \
    && cd .. \
    && rm -rf qt6

# Get linuxdeploy and appimagetool
RUN apt -y install wget
RUN wget -c https://github.com/AppImage/appimagetool/releases/download/continuous/appimagetool-x86_64.AppImage \
        && chmod +x appimagetool-x86_64.AppImage \
        && ./appimagetool-x86_64.AppImage --appimage-extract \
        && cp -R squashfs-root/usr/* /usr/local \
        && rm -r appimagetool-x86_64.AppImage squashfs-root
RUN wget -c https://github.com/linuxdeploy/linuxdeploy-plugin-qt/releases/download/1-alpha-20240109-1/linuxdeploy-plugin-qt-x86_64.AppImage \
        && chmod +x linuxdeploy-plugin-qt-x86_64.AppImage \
        && ./linuxdeploy-plugin-qt-x86_64.AppImage --appimage-extract \
        && cp -R squashfs-root/usr/* /usr/local \
        && rm -r linuxdeploy-plugin-qt-x86_64.AppImage squashfs-root
RUN wget -c https://github.com/linuxdeploy/linuxdeploy/releases/download/1-alpha-20240109-1/linuxdeploy-x86_64.AppImage \
        && chmod +x linuxdeploy-x86_64.AppImage \
        && ./linuxdeploy-x86_64.AppImage --appimage-extract \
        && cp -R squashfs-root/usr/* /usr/local \
        && rm -r linuxdeploy-x86_64.AppImage squashfs-root

# Other dependencies for Vanilla/AppImages
RUN apt -y install libavcodec-dev libavfilter-dev libavutil-dev \
        libsdl2-dev libnl-genl-3-dev libssl-dev desktop-file-utils

# Install latest CMake
RUN wget -c https://github.com/Kitware/CMake/releases/download/v3.29.3/cmake-3.29.3-linux-x86_64.sh \
        && chmod +x cmake-3.29.3-linux-x86_64.sh \
        && ./cmake-3.29.3-linux-x86_64.sh --skip-license --prefix=/usr/local \
        && rm cmake-3.29.3-linux-x86_64.sh

# Build Vanilla and make an AppImage
CMD git config --global --add safe.directory /vanilla/server/hostap \
        && cp /vanilla/server/hostap/conf/wpa_supplicant.config /vanilla/server/hostap/wpa_supplicant/.config \
        && /usr/local/bin/cmake /vanilla -B/build -G Ninja -DCMAKE_INSTALL_PREFIX=/AppDir/usr \
        && cmake --build /build \
        && cmake --install /build \
        && linuxdeploy --appdir=/AppDir --plugin qt \
        && appimagetool /AppDir \
        && mv Vanilla*.AppImage /vanilla