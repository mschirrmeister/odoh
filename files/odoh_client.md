# ODoH client

In order to use **ODoH** you need a client/resolver that supports ODoH. The ones that I found so far, are the following.

- [dnscrypt-proxy](https://github.com/DNSCrypt/dnscrypt-proxy)
- [RouteDNS](https://github.com/folbricht/routedns)

The problem is that ODoH is still a work in progress and no final standard yet. That means that clients and servers might be incompatible for certain features.  
For example the encryption keys can be retrieves from an **ODoH target** via DNS type `type65` or via an HTTP request URI `/.well-known/odohconfigs`. RouteDNS does not support getting the keys via an http request and **Cloudflare** does no longer provide the keys via **DNS**. Thus, RouteDNS cannot be used if you want to use Cloudflares ODoH.

Most available ODoH target servers that are available at this point, provide the encryption keys via HTTP. **dnscrypt-proxy** is therefore a good choice at the moment for a client that you want to run at home.

### dnscrypt-proxy

Running dnscrypt-proxy via Docker

Build in config

    docker run -it -d \
      --name odoh-resolver \
      -p 53:53/tcp \
      -p 53:53/udp \
      -v /Users/marco/MyData/git/public/odoh-resolver-docker/odoh-proxied.toml:/config/odoh-proxied.toml \
      mschirrmeister/odoh-resolver:latest -loglevel 1 -config /config/odoh-proxied.toml -pidfile /var/run/odoh-proxied.pidfile

Custom config. You can find an example **custom** config in my github repo https://github.com/mschirrmeister/odoh-resolver-docker

    docker run -it -d \
      --name odoh-resolver \
      -p 53:53/tcp \
      -p 53:53/udp \
      -v /Users/marco/MyData/git/public/odoh-resolver-docker/odoh-proxied.toml:/config/odoh-proxied.toml \
      mschirrmeister/odoh-resolver:latest -loglevel 0 -config /config/odoh-proxied.toml -pidfile /var/run/odoh-proxied.pidfile

## testing final odoh resolution

    dig a cloudflare-dns.com @127.0.0.1
    dig aaaa cloudflare-dns.com @127.0.0.1
