FROM archlinux:base-devel AS base

ARG USERNAME=appuser
ARG USER_UID=1000
ARG USER_GID=$USER_UID


RUN pacman -Syu --noconfirm

# Install dependancies needed by all steps including runtime step
RUN pacman -S --noconfirm --needed aom ffmpeg vapoursynth ffms2 libvpx mkvtoolnix-cli svt-av1 vapoursynth-plugin-lsmashsource vmaf

# Install tools
RUN pacman -S --noconfirm --needed perf sudo 

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

USER $USERNAME

FROM base AS build-base

# Install dependancies needed by build steps
RUN sudo pacman -S --noconfirm --needed rust clang nasm git 

RUN cargo install cargo-chef
WORKDIR /tmp/Av1an


FROM build-base AS planner

COPY . .
RUN cargo chef prepare


FROM build-base AS build

COPY --from=planner /tmp/Av1an/recipe.json recipe.json
RUN cargo chef cook --release

# Compile rav1e from git, as archlinux is still on rav1e 0.4
RUN git clone https://github.com/xiph/rav1e && \
    cd rav1e && \
    cargo build --release && \
    strip ./target/release/rav1e && \
    sudo mv ./target/release/rav1e /usr/local/bin && \
    cd .. && rm -rf ./rav1e
