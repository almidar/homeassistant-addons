FROM ghcr.io/home-assistant/amd64-base-debian:bookworm

LABEL io.hass.version="1.0" io.hass.type="addon" io.hass.arch="amd64"

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        sudo \
        locales \
        cups \
        avahi-daemon \
        libnss-mdns \
        dbus \
        colord \
        printer-driver-all-enforce \
        openprinting-ppds \
        cups-pdf \
        gnupg2 \
        lsb-release \
        nano \
        samba \
        bash-completion \
        procps \
        whois \
        build-essential \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

COPY brhl2240cups_src-2.0.4-2.tar.gz /
COPY cupswrapperHL2240-2.0.4-2.i386.tar.gz /
COPY hl2240lpr-2.1.0-1.i386.tar.gz /

# Repackage LPR drivers
RUN tar -zxvf hl2240lpr-2.1.0-1.i386.tar.gz \
    && dpkg -x hl2240lpr-2.1.0-1.i386.deb hl2240lpr-2.1.0-1.amd64.extracted \
    && dpkg-deb -e hl2240lpr-2.1.0-1.i386.deb hl2240lpr-2.1.0-1.amd64.extracted/DEBIAN \
    && sed -i 's/Architecture: i386/Architecture: amd64/' hl2240lpr-2.1.0-1.amd64.extracted/DEBIAN/control \
    && echo true > hl2240lpr-2.1.0-1.amd64.extracted/usr/local/Brother/Printer/HL2240/inf/braddprinter

RUN cd hl2240lpr-2.1.0-1.amd64.extracted \
    && find . -type f ! -regex '.*.hg.*' ! -regex '.*?debian-binary.*' ! -regex '.*?DEBIAN.*' -printf '%P ' | xargs md5sum > DEBIAN/md5sums \
    && cd .. \
    && chmod 755 hl2240lpr-2.1.0-1.amd64.extracted/DEBIAN/p* hl2240lpr-2.1.0-1.amd64.extracted/usr/local/Brother/Printer/HL2240/inf/* hl2240lpr-2.1.0-1.amd64.extracted/usr/local/Brother/Printer/HL2240/lpd/* \
    && dpkg-deb -b hl2240lpr-2.1.0-1.amd64.extracted hl2240lpr-2.1.0-1.amd64.deb

# build cups wrapper
RUN tar -zxvf brhl2240cups_src-2.0.4-2.tar.gz
RUN cd brhl2240cups_src-2.0.4-2 \
    && gcc brcupsconfig3/brcupsconfig.c -o brcupsconfig4 \
    && cd ..

# extract i386 cupswrapper
RUN tar -zxvf cupswrapperHL2240-2.0.4-2.i386.tar.gz \
    && dpkg -x cupswrapperHL2240-2.0.4-2.i386.deb cupswrapperHL2240-2.0.4-2.amd64.extracted \
    && dpkg-deb -e cupswrapperHL2240-2.0.4-2.i386.deb cupswrapperHL2240-2.0.4-2.amd64.extracted/DEBIAN \
    && sed -i 's/Architecture: i386/Architecture: amd64/' cupswrapperHL2240-2.0.4-2.amd64.extracted/DEBIAN/control

# copy build wrapper to extracted deb    
RUN cp brhl2240cups_src-2.0.4-2/brcupsconfig4 cupswrapperHL2240-2.0.4-2.amd64.extracted/usr/local/Brother/Printer/HL2240/cupswrapper

# repack deb
RUN cd cupswrapperHL2240-2.0.4-2.amd64.extracted \
    && find . -type f ! -regex '.*.hg.*' ! -regex '.*?debian-binary.*' ! -regex '.*?DEBIAN.*' -printf '%P ' | xargs md5sum > DEBIAN/md5sums \
    && cd .. \
    && chmod 755 cupswrapperHL2240-2.0.4-2.amd64.extracted/DEBIAN/p* cupswrapperHL2240-2.0.4-2.amd64.extracted/usr/local/Brother/Printer/HL2240/cupswrapper/* \
    && dpkg-deb -b cupswrapperHL2240-2.0.4-2.amd64.extracted cupswrapperHL2240-2.0.4-2.amd64.deb
RUN cd /

RUN apt-get update \
    && apt install ./hl2240lpr-2.1.0-1.amd64.deb \
    && apt install ./cupswrapperHL2240-2.0.4-2.amd64.deb \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

COPY rootfs /

# Add user and disable sudo password checking
RUN useradd \
  --groups=sudo,lp,lpadmin \
  --create-home \
  --home-dir=/home/print \
  --shell=/bin/bash \
  --password=$(mkpasswd print) \
  print \
&& sed -i '/%sudo[[:space:]]/ s/ALL[[:space:]]*$/NOPASSWD:ALL/' /etc/sudoers

EXPOSE 631

RUN chmod a+x /run.sh

CMD ["/run.sh"]
