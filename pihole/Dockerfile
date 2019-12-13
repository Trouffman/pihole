FROM balenalib/raspberrypi3-debian:jessie-build AS build

WORKDIR /usr/src

RUN install_packages cmake libraspberrypi-dev

RUN curl -sSL https://raw.githubusercontent.com/tasanakorn/rpi-fbcp/master/CMakeLists.txt -O
RUN curl -sSL https://raw.githubusercontent.com/tasanakorn/rpi-fbcp/master/main.c -O

WORKDIR /usr/src/build

RUN cmake .. && make

FROM pihole/pihole:4.3.2-1_armhf

WORKDIR /usr/src

COPY --from=build /opt/vc/lib/* /opt/vc/lib/
COPY --from=build /usr/src/build/fbcp /usr/src/
COPY services/ /etc/services.d/
COPY init/ /etc/cont-init.d/

RUN sed -i '/$AUTHORIZED_HOSTNAMES = array(/ a "balena-devices.com",' /var/www/html/admin/scripts/pi-hole/php/auth.php

# https://serverfault.com/a/817791
# force dnsmasq to really bind only the interfaces it is listening on
# otherwise dnsmasq will fail to start since balena is using 53 on some interfaces
RUN echo "bind-interfaces" >> /etc/dnsmasq.conf

# Force DHCP to be ON at first boot (might cause conflict with other router / DHCP server)
# RUN echo "# File generated at Container Creation and NOT by the web interface (like it should usually)"  >> 02-pihole-dhcp.conf
# Copy the config to activate DHCP at container creation
COPY conf/ /etc/dnsmasq.d/


RUN curl -fsSL https://github.com/jpmck/PADD/archive/v3.0.2.tar.gz | tar xz --strip-components 1 \
	&& chmod +x padd.sh

# use wally3k's ticked list from https://firebog.net/
RUN curl -sSL https://v.firebog.net/hosts/lists.php?type=tick -o /etc/pihole/adlists.list
