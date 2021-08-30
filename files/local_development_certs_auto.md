# Create a local ODoH setup

Here are the steps to setup a local ODoH environment. You need to adjust server names, directories, filenames.

## Do53 resolver

Get ip address of local interface

    # fish
    set IP (ip a s en0 | ggrep -oP '(?<=inet\s)\d+(\.\d+){3}')
    # bash
    IP=$(ip a s en0 |  grep -oP '(?<=inet\s)\d+(\.\d+){3}')

Run knot resolver to listen on 53/tcp/udp

    docker run -it -d \
      --name knot-resolver \
      cznic/knot-resolver

## doh-server

Example on how to build the `doh-server`

    git clone https://github.com/jedisct1/doh-server.git
    cd doh-server
    # get latest commit hash
    git --git-dir /Users/marco/mydata/git/public/doh-server/.git log -1 --format=%h

    docker build -t doh-server:0.9.0-f9d2a0f .
    # or
    docker build -t doh-server:0.9.0-(git --git-dir /Users/marco/mydata/git/public/doh-server/.git log -1 --format=%h) .
    # or
    docker build -t doh-server:latest

Check version
 
    docker run -it --rm --name odoh-server \
      doh-server:latest --version

Run container locally for testing

    docker run -it -d \
      --name odoh-server-target \
      -p 3000:3000/tcp \
      -v /Users/marco/MyData/Secure/Certificates/testing/mycerts/doh.marco.cx/doh.marco.cx.key:/certs/cert.key \
      -v /Users/marco/MyData/Secure/Certificates/testing/mycerts/doh.marco.cx/fullchain.cer:/certs/cert.pem \
      --link knot-resolver:knot-resolver \
      mschirrmeister/doh-server:0.9.0-f9d2a0f --tls-cert-key-path /certs/cert.key --tls-cert-path /certs/cert.pem -l 0.0.0.0:3000 --server-address knot-resolver:53 -H doh.marco.cx --allow-odoh-post

Run container locally and register in traefik with **https** to the backend

    docker run -it -d \
      --name odoh-server-target \
      -p 127.0.0.1:3000:3000/tcp \
      -v /Users/marco/MyData/Secure/Certificates/testing/mycerts/doh.marco.cx/doh.marco.cx.key:/certs/cert.key \
      -v /Users/marco/MyData/Secure/Certificates/testing/mycerts/doh.marco.cx/fullchain.cer:/certs/cert.pem \
      --link knot-resolver:knot-resolver \
      -l "traefik.http.routers.odohtarget.rule=Host(`doh.marco.cx`) || Host(`odoh-target.marco.cx`) || Host(`odoh-t-fra.marco.cx`) || Host(`odoh-target-fra.marco.cx`)' \
      -l "traefik.http.routers.odohtarget.tls=true" \
      -l "traefik.http.routers.odohtarget.tls.certresolver=le" \
      -l "traefik.http.routers.odohtarget.tls.options=mintls12@file" \
      -l "traefik.http.routers.odohtarget.tls.domains[0].main=doh.marco.cx" \
      -l "traefik.http.routers.odohtarget.tls.domains[0].sans=odoh-target.marco.cx" \
      -l "traefik.http.services.odohtarget.loadbalancer.server.scheme=https" \
      -l "traefik.http.services.odohtarget.loadbalancer.serversTransport=odohtarget@file" \
      doh-server:0.9.0-f9d2a0f --tls-cert-key-path /certs/cert.key --tls-cert-path /certs/cert.pem -l 0.0.0.0:3000 --server-address knot-resolver:53 -H doh.marco.cx --allow-odoh-post

Run container locally and register in traefik with **http** to the backend

    docker run -it -d \
      --name odoh-server-target \
      -p 127.0.0.1:3000:3000/tcp \
      --link knot-resolver:knot-resolver \
      -l 'traefik.http.routers.odohtarget.rule=Host(`doh.marco.cx`) || Host(`odoh-target.marco.cx`) || Host(`odoh-t-fra.marco.cx`) || Host(`odoh-target-fra.marco.cx`)' \
      -l 'traefik.http.routers.odohtarget.tls=true' \
      -l "traefik.http.routers.odohtarget.tls.certresolver=le" \
      -l "traefik.http.routers.odohtarget.tls.options=mintls12@file" \
      -l "traefik.http.routers.odohtarget.tls.domains[0].main=doh.marco.cx" \
      -l "traefik.http.routers.odohtarget.tls.domains[0].sans=odoh-target.marco.cx" \
      mschirrmeister/doh-server:0.9.0-f9d2a0f -l 0.0.0.0:3000 --server-address knot-resolver:53 -H doh.marco.cx --allow-odoh-post

### doh-server testing

    curl -H 'Host: doh.marco.cx' -H 'Accept: application/dns-message' --resolve doh.marco.cx:3000:127.0.0.1 'https://doh.marco.cx:3000/dns-query?dns=q80BAAABAAAAAAAAA3d3dwdleGFtcGxlA2NvbQAAAQAB' | hexdump -c
    curl -k -H 'accept: application/dns-message' 'https://127.0.0.1:3000/dns-query?dns=q80BAAABAAAAAAAAA3d3dwdleGFtcGxlA2NvbQAAAQAB'  | hexdump -cb
    kdig @127.0.0.1 -p 3000 +https example.com
    odoh-client odoh --domain www.cloudflare.com. --dnstype AAAA --target http://doh.marco.cx:3000
    odoh-client odohconfig-fetch --target http://doh.marco.cx:3000 --pretty

## odoh-server-go

Get ip address of local interface

    # fish
    set IP (ip a s en0 | ggrep -oP '(?<=inet\s)\d+(\.\d+){3}')
    # bash
    IP=$(ip a s en0 |  grep -oP '(?<=inet\s)\d+(\.\d+){3}')

Example on how to build the `odoh-server-go`

    git --git-dir /Users/marco/mydata/git/public/odoh-server-go/.git log -1 --format=%h
    docker build -t odoh-server-go:0.1.0-7986d2f .
    docker build -t odoh-server-go:0.1.0-(git --git-dir /Users/marco/mydata/git/public/odoh-server-go/.git log -1 --format=%h) .

Example on how to load a local Docker image to a server

    # local
    docker save -o ~/tmp/odoh-serve-go.tar.gz odoh-server-go:0.1.4-7986d2f
    # cloud server
    docker load -i doh-server-go.tar.gz

or

    docker save odoh-server-go:0.1.4-7986d2f | bzip2 | pv | ssh root@host: 'bunzip2 | docker load'

Run container with build-in **https** support

    docker run -it -d \
      --name odoh-server-relay \
      -p 4567:4567/tcp \
      -v /Users/marco/MyData/Secure/Certificates/testing/mycerts/doh.marco.cx_ecc:/certs \
      -e TARGET_INSTANCE_NAME=doh.marco.cx \
      -e SEED_SECRET_KEY=(openssl rand -hex 16) \
      -e EXPERIMENT_ID=EXP_1 \
      -e CERT=/certs/fullchain.cer \
      -e KEY=/certs/doh.marco.cx.key \
      -e PORT=4567 \
      mschirrmeister/odoh-server-go:0.1.4-5a9bf1f

Run container locally and register in traefik with **https** to the backend

    docker run -it -d \
      --name odoh-server-relay \
      -p 127.0.0.1:4567:4567/tcp \
      -v /Users/marco/MyData/Secure/Certificates/testing/mycerts/doh.marco.cx:/certs \
      -e TARGET_INSTANCE_NAME=odoh.marco.cx \
      -e SEED_SECRET_KEY=(openssl rand -hex 16) \
      -e EXPERIMENT_ID=EXP_1 \
      -e CERT=/certs/fullchain.cer \
      -e KEY=/certs/doh.marco.cx.key \
      -e PORT=4567 \
      --add-host doh.marco.cx:192.168.42.212 \
      -l "traefik.http.routers.odohproxy.rule=Host(`odoh.marco.cx`) || Host(`odoh-proxy.marco.cx`) || Host(`odoh-relay.marco.cx`) || Host(`odoh-p-fra.marco.cx`) || Host(`odoh-proxy-fra.marco.cx`) || Host(`odoh-r-fra.marco.cx`) || Host(`odoh-relay-fra.marco.cx`)" \
      -l "traefik.http.routers.odohproxy.tls=true" \
      -l "traefik.http.routers.odohtarget.tls.certresolver=le" \
      -l "traefik.http.routers.odohtarget.tls.options=mintls12@file" \
      -l "traefik.http.routers.odohtarget.tls.domains[0].main=odoh.marco.cx" \
      -l "traefik.http.routers.odohtarget.tls.domains[0].sans=odoh-relay.marco.cx" \
      -l "traefik.http.services.odohproxy.loadbalancer.server.scheme=https" \
      -l "traefik.http.services.odohproxy.loadbalancer.serversTransport=odohproxy@file" \
      mschirrmeister/odoh-server-go:0.1.4-5a9bf1f

Run container locally and register in traefik with **http** to the backend

    docker run -it -d \
      --name odoh-server-relay \
      -p 127.0.0.1:4567:4567/tcp \
      -e TARGET_INSTANCE_NAME=odoh.marco.cx \
      -e SEED_SECRET_KEY=(openssl rand -hex 16) \
      -e EXPERIMENT_ID=EXP_1 \
      -e PORT=4567 \
      --add-host doh.marco.cx:$IP \
      -l 'traefik.http.routers.odohproxy.rule=Host(`odoh.marco.cx`) || Host(`odoh-proxy.marco.cx`) || Host(`odoh-relay.marco.cx`) || Host(`odoh-p-fra.marco.cx`) || Host(`odoh-proxy-fra.marco.cx`) || Host(`odoh-r-fra.marco.cx`) || Host(`odoh-relay-fra.marco.cx`)" \
      -l 'traefik.http.routers.odohproxy.tls=true' \
      -l "traefik.http.routers.odohtarget.tls.certresolver=le" \
      -l "traefik.http.routers.odohtarget.tls.options=mintls12@file" \
      -l "traefik.http.routers.odohtarget.tls.domains[0].main=odoh.marco.cx" \
      -l "traefik.http.routers.odohtarget.tls.domains[0].sans=odoh-relay.marco.cx" \
      mschirrmeister/odoh-server-go:0.1.4-5a9bf1f

### odoh-server-go testing

You can test odoh with the `odoh-client`. The following example can be used if the containers above have been started without traefik where the ports are directy expost. You also need hosts entries.

    ./odoh-client odoh --domain www.cloudflare.com. --dnstype AAAA --target doh.marco.cx:3000 --proxy doh.marco.cx:4567

## Traefik

Traefik config examples

- <https://www.digitalocean.com/community/tutorials/how-to-use-traefik-v2-as-a-reverse-proxy-for-docker-containers-on-ubuntu-20-04>
- <https://github.com/DoTheEvo/Traefik-v2-examples>

Run Traefik locally with existing static certificate

    docker run -it -d \
      --name traefik \
      -p 8080:8080 \
      -p 80:80 \
      -p 443:443 \
      -e CLOUDNS_AUTH_ID=foo \
      -e CLOUDNS_AUTH_PASSWORD=bar \
      --env-file /Users/marco/MyData/git/public/odoh/.env \
      -v /Users/marco/MyData/git/public/odoh/traefik/traefik.yml:/etc/traefik/traefik.yml \
      -v /Users/marco/MyData/git/public/odoh/traefik/certs:/letsencrypt \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v /Users/marco/MyData/Secure/Certificates/testing/mycerts/doh.marco.cx/:/certs1/ \
      -v /Users/marco/MyData/Secure/Certificates/testing/mycerts/doh.marco.cx_ecc/:/certs2/ \
      -v /Users/marco/MyData/git/public/odoh/traefik/dynamic/:/etc/traefik/dynamic/ \
      traefik:v2.4.13

### odoh testing

    odoh-client odoh --domain www.cloudflare.com. --dnstype AAAA --target doh.marco.cx --proxy doh.marco.cx
    
## dnscrypt-proxy

Get ip address of local interface

    # fish
    set IP (ip a s en0 | ggrep -oP '(?<=inet\s)\d+(\.\d+){3}')
    # bash
    IP=$(ip a s en0 |  grep -oP '(?<=inet\s)\d+(\.\d+){3}')

Running dnscrypt-proxy via Docker

    docker run -it -d \
      --name odoh-resolver \
      -p 53:53/tcp \
      -p 53:53/udp \
      -v /Users/marco/MyData/git/public/odoh-resolver-docker/odoh-proxied.toml:/config/odoh-proxied.toml \
      --add-host doh.marco.cx:$IP \
      --add-host odoh.marco.cx:$IP \
      odoh-resolver:latest -loglevel 1 -config /config/odoh-proxied.toml -pidfile /var/run/odoh-proxied.pidfile

## testing final odoh resolution

    dig a example.com @127.0.0.1
    dig aaaa example.com @127.0.0.1
    q a aaaa example.com @127.0.0.1

Few more examples for Do53, DoT, DoH, DoQ and ODoH testing with the [q](https://github.com/natesales/q) client.

    # Do53
    q a aaaa example.com @1.1.1.1
    # DoT
    q a aaaa example.com @tls://1.1.1.1:853
    # DoH
    q a aaaa example.com @https://cloudflare-dns.com/dns-query
    q a aaaa example.com @https://1.1.1.1/dns-query
    # DoQ
    q a aaaa example.com @quic://dns.adguard.com:8853
    # ODoH
    q a aaaa example.com --server=https://odoh.cloudflare-dns.com --odoh-proxy=https://odoh1.surfdomeinen.nl
    q a aaaa example.com --server=https://odoh-target.marco.cx --odoh-proxy=https://odoh-relay.marco.cx
