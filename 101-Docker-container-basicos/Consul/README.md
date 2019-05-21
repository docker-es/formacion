docker run --restart=unless-stopped -d -h consul0 --name consul0 -v /mnt:/data \
        -p $(hostname -i):8300:8300 \
        -p $(hostname -i):8301:8301 \
        -p $(hostname -i):8301:8301/udp \
        -p $(hostname -i):8302:8302 \
        -p $(hostname -i):8302:8302/udp \
        -p $(hostname -i):8400:8400 \
        -p $(hostname -i):8500:8500 \
        -p $(ifconfig docker0 | awk '/\<inet\>/ { print $2}' | cut -d: -f2):53:53/udp \
        progrium/consul -server -advertise $(hostname -i) -bootstrap-expect 3


otro
docker run -d -p 8400:8400 -p 8500:8500 -p 53:53/udp -h consul-server-node --name consul progrium/consul -server -bootstrap
docker run -v $(pwd):/mnt


PORTAINER
git  clone https://github.com/portainer/portainer-compose.git
para acceder: http://localhost/portainer 