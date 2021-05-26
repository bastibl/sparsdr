FROM ubuntu:18.04

LABEL maintainers="Bastian Bloessl <mail@bastibl.net>, Florian Eckert <florian.eckert@ymail.com>"

ARG USER=gnuradio

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get install -y git vim wget unzip sudo cmake libtool build-essential pkg-config autogen python-dev python-six swig3.0 python3-mako python3-numpy gnuradio libuhd-dev libfftw3-3 curl libqt5widgets5 libqwt-qt5-dev qt5-default g++ libboost-all-dev libgmp-dev swig python3-sphinx python3-lxml doxygen libfftw3-dev libsdl1.2-dev libgsl-dev libqt5opengl5-dev python3-pyqt5 liblog4cpp5-dev libzmq3-dev python3-yaml python3-click python3-click-plugins  python3-zmq python3-scipy

RUN groupadd -g 1000 -r $USER
RUN useradd -u 1000 -g 1000 --create-home -r $USER

RUN echo "$USER:$USER" | chpasswd

#Make sudo passwordless
RUN mkdir -p /etc/sudoers.d
RUN echo "${USER} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-$USER
RUN usermod -aG sudo $USER
RUN usermod -aG plugdev $USER

USER $USER
ENV HOME /home/$USER

# Get Rust
RUN curl https://sh.rustup.rs -sSf | bash -s -- -y
RUN echo 'source $HOME/.cargo/env' >> $HOME/.bashrc

RUN mkdir -p $HOME/src
WORKDIR $HOME/src
RUN git clone  https://github.com/bastibl/sparsdr.git --branch maint-3.7
RUN mkdir -p $HOME/src/sparsdr/gr-sparsdr/build
WORKDIR $HOME/src/sparsdr/gr-sparsdr/build
RUN cmake ..
RUN make
RUN sudo make install
RUN sudo ldconfig

ENV PATH $HOME/.cargo/bin:$PATH
WORKDIR $HOME/src/sparsdr/reconstruct
RUN cargo install --path .
RUN cargo test

RUN sudo uhd_images_downloader

RUN sudo sed -i 's/style = raster/style = native/g' /etc/gnuradio/conf.d/gr-qtgui.conf

RUN sudo apt-get install -y libbtbb-dev
WORKDIR $HOME/src
RUN git clone https://github.com/greatscottgadgets/gr-bluetooth
RUN mkdir -p $HOME/src/gr-bluetooth/build
WORKDIR $HOME/src/gr-bluetooth/build
RUN cmake ..
RUN make
RUN sudo make install
RUN sudo ldconfig

WORKDIR $HOME
ENTRYPOINT ["/bin/bash"]
