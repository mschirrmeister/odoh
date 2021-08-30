# Create a public ODoH setup

Here are the steps to setup a local ODoH environment. You need to adjust server names, directories, filenames.

## Digitialocean

### Do53 resolver

Get public ip address

    # fish
    set IP (ip a s eth1 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
    # bash
    IP=$(ip a s eth1 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')

Run knot resolver to listen on 53/tcp/udp

    docker run -it -d \
      --name knot-resolver \
      -p ${IP}:53:53/udp \
      -p ${IP}:53:53/tcp \
      cznic/knot-resolver

### doh-server

Get internal ip address from cloud server

    # fish
    set IP (ip a s eth1 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
    # bash
    IP=$(ip a s eth1 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')

Run container and register in Traefik

    docker run -it -d \
      --name odoh-server-target \
      -p 127.0.0.1:3000:3000/tcp \
      -p 127.0.0.1:3000:3000/udp \
      --link knot-resolver:knot-resolver \
      -l 'traefik.http.routers.odohtarget.rule=Host(`doh.marco.cx`) || Host(`odoh-target.marco.cx`) || Host(`odoh-t-fra.marco.cx`) || Host(`odoh-target-fra.marco.cx`)' \
      -l 'traefik.http.routers.odohtarget.tls=true' \
      -l "traefik.http.routers.odohtarget.tls.certresolver=le" \
      -l "traefik.http.routers.odohtarget.tls.options=mintls12@file" \
      -l "traefik.http.routers.odohtarget.tls.domains[0].main=doh.marco.cx" \
      -l "traefik.http.routers.odohtarget.tls.domains[0].sans=odoh-target.marco.cx" \
      mschirrmeister/doh-server:0.9.0-f9d2a0f -l 0.0.0.0:3000 --server-address knot-resolver:53 -H doh.marco.cx --allow-odoh-post

### Traefik

Run Traefik on a public cloud server with certificates managed by Traefik

    docker run -it -d \
      --name traefik \
      -p 127.0.0.1:8080:8080 \
      -p 80:80 \
      -p 443:443 \
      --env-file /home/marco/.env \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v /home/marco/traefik/traefik.yml:/etc/traefik/traefik.yml \
      -v /home/marco/traefik/dynamic/:/etc/traefik/dynamic/ \
      -v /home/marco/traefik/certs:/letsencrypt \
      -v /home/marco/doh.marco.cx/:/certs1/ \
      -v /home/marco/doh.marco.cx_ecc/:/certs2/ \
      traefik:v2.4.13

## Vultr

### odoh-server-go

Get public ip address

    # fish
    set IP (ip a s enp1s0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
    # bash
    IP=$(ip a s enp1s0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')

Run container and register in traefik

    docker run -it -d \
      --name odoh-server-relay \
      -p 127.0.0.1:4567:4567/tcp \
      -e TARGET_INSTANCE_NAME=odoh.marco.cx \
      -e SEED_SECRET_KEY=$(openssl rand -hex 16) \
      -e EXPERIMENT_ID=EXP_1 \
      -e PORT=4567 \
      -l 'traefik.http.routers.odohproxy.rule=Host(`odoh.marco.cx`) || Host(`odoh-proxy.marco.cx`) || Host(`odoh-relay.marco.cx`) || Host(`odoh-p-fra.marco.cx`) || Host(`odoh-proxy-fra.marco.cx`) || Host(`odoh-r-fra.marco.cx`) || Host(`odoh-relay-fra.marco.cx`)' \
      -l 'traefik.http.routers.odohproxy.tls=true' \
      -l "traefik.http.routers.odohproxy.tls.certresolver=le" \
      -l "traefik.http.routers.odohproxy.tls.options=mintls12@file" \
      -l "traefik.http.routers.odohproxy.tls.domains[0].main=odoh.marco.cx" \
      -l "traefik.http.routers.odohproxy.tls.domains[0].sans=odoh-relay.marco.cx" \
      mschirrmeister/odoh-server-go:0.1.4-5a9bf1f

### Traefik

Run Traefik on a public cloud server with certificates managed by Traefik

    docker run -it -d \
      --name traefik \
      -p 127.0.0.1:8080:8080 \
      -p 80:80 \
      -p 443:443 \
      --env-file /home/marco/.env \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v /home/marco/traefik/traefik.yml:/etc/traefik/traefik.yml \
      -v /home/marco/traefik/dynamic/:/etc/traefik/dynamic/ \
      -v /home/marco/traefik/certs:/letsencrypt \
      -v /home/marco/doh.marco.cx/:/certs1/ \
      -v /home/marco/doh.marco.cx_ecc/:/certs2/ \
      traefik:v2.4.13
