ARG BASE_IMAGE=sickcodes/docker-osx:latest
FROM ${BASE_IMAGE}

MAINTAINER 'https://twitter.com/sickcodes' <https://sick.codes>

USER root

ARG RANKMIRRORS
ARG MIRROR_COUNTRY=US
ARG MIRROR_COUNT=10
RUN if [[ "${RANKMIRRORS}" ]]; then { pacman -Sy wget --noconfirm || pacman -Syu wget --noconfirm ; } \
    ; wget -O ./rankmirrors "https://raw.githubusercontent.com/sickcodes/Docker-OSX/master/rankmirrors" \
    ; wget -O- "https://www.archlinux.org/mirrorlist/?country=${MIRROR_COUNTRY:-US}&protocol=https&use_mirror_status=on" \
    | sed -e 's/^#Server/Server/' -e '/^#/d' \
    | head -n "$((${MIRROR_COUNT:-10}+1))" \
    | bash ./rankmirrors --verbose --max-time 5 - > /etc/pacman.d/mirrorlist \
    && tee -a /etc/pacman.d/mirrorlist <<< 'Server = http://mirrors.evowise.com/archlinux/$repo/os/$arch' \
    && tee -a /etc/pacman.d/mirrorlist <<< 'Server = http://mirror.rackspace.com/archlinux/$repo/os/$arch' \
    && tee -a /etc/pacman.d/mirrorlist <<< 'Server = https://mirror.rackspace.com/archlinux/$repo/os/$arch' \
    && cat /etc/pacman.d/mirrorlist ; fi

RUN pacman -Sy archlinux-keyring --noconfirm && rm -rf /etc/pacman.d/gnupg && pacman-key --init && pacman-key --populate

USER arch

RUN yes | sudo pacman -Syyuu --noconfirm \
    && yes | sudo pacman -S tigervnc xterm xorg-xhost xdotool ufw --noconfirm \
    && mkdir -p ${HOME}/.vnc \
    && touch ~/.vnc/config \
    && tee -a ~/.vnc/config <<< 'geometry=1920x1080' \
    && tee -a ~/.vnc/config <<< 'localhost' \
    && tee -a ~/.vnc/config <<< 'alwaysshared'

RUN printf '\n%s\n' \
'sudo rm -f /tmp/.X99-lock' \
'export DISPLAY=:99' \
'/usr/bin/Xvnc -geometry 1920x1080 -rfbauth "${HOME}/.vnc/passwd" :99 &' > vnc.sh

RUN cat vnc.sh Launch.sh > Launch_custom.sh

RUN chmod +x Launch_custom.sh

RUN tee vncpasswd_file <<< "${VNC_PASSWORD:="$(tr -dc '[:graph:]' </dev/urandom | head -c8)"}"
RUN vncpasswd -f < vncpasswd_file > ${HOME}/.vnc/passwd

RUN chmod 600 ~/.vnc/passwd
RUN printf '\n\n\n\n%s\n%s\n\n\n\n' '===========VNC_PASSWORD========== ' "$(<vncpasswd_file)"

WORKDIR /home/arch/OSX-KVM

CMD ./enable-ssh.sh && envsubst < ./Launch_custom.sh | bash
