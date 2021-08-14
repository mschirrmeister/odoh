# Oblivious DNS-Over-HTTPS (ODoH)

**ODoH** is a new DNS protocol that is privacy focused. It ensures that the service providers can no longer associate the the dns requests with the user. **Cloudflare** has a pretty good blog about **ODoH**. <https://blog.cloudflare.com/oblivious-dns/>

**ODoH** is no standard yet, but everything to try it out is already available. That means everyone can set it up and use it for either just getting to know it and learn something, or use it in order to do real privacy dns resolution.

This repo contains information how you can setup ODoH to run it and learn it.

## ODoH components

For ODoH you have 2 main components.

- ODoH Target
- ODoH Relay

The **Target** is the dns server that will do the final dns resolution.  
The **Relay** is like a proxy to which the users **odoh** capable resolver will talk to.

The client will also talk to the **target**, but only **one** time at the beginning, to retrieve the **encryption key**. Some servers had implementations where the key could be retrieved via a dns query via the odoh relay (DNSSEC). At the moment, no server I have seen implements it anymore.

## Notes/thoughts on security

Retrieving the encryption key via a dns query seems to be more secure, because the query would also come via the relay/proxy and not directly from the client like with a http request directly from client to target.  
If the client talks directly with the odoh target via http, there is the argument that the odoh target server could theoretically provide encryption keys unique to the client, which means the anonymity is gone, because the server can correlate the queries to a client when the final requests come via the proxy.

As long as the ODoH servers can be trusted, this should not be a problem, with the assumption they don't do bad things.

## Certificates

For testing purposes, certificates are created manually. Here is an example on how to get a certificate from **LetsEncrypt** with the `acme.sh` tool. You don't need this if you use the guide where the certificates are automated via **Traefik**.

    # ECC
    acme.sh --issue --dns dns_cloudns --dnssleep 120 -d doh.marco.cx -d odoh.marco.cx -d odoh-p-fra.marco.cx -d odoh-r-fra.marco.cx -d odoh-t-fra.marco.cx -d odoh-proxy-fra.marco.cx -d odoh-relay-fra.marco.cx -d odoh-target-fra.marco.cx --server letsencrypt --keylength ec-256 --ocsp
    # RSA
    acme.sh --issue --dns dns_cloudns --dnssleep 120 -d doh.marco.cx -d odoh.marco.cx -d odoh-p-fra.marco.cx -d odoh-r-fra.marco.cx -d odoh-t-fra.marco.cx -d odoh-proxy-fra.marco.cx -d odoh-relay-fra.marco.cx -d odoh-target-fra.marco.cx --server letsencrypt --keylength 3072 --ocsp

## Setup guide

This guide should help you in setting up an ODoH environment to get started and to learn about ODoH and how it works. The guide is based on Docker containers. All commands to run each component are pure Docker commands. No docker-compose or any other service automation for now. Reason is to keep it very simple for the beginning. The guide might get extended to more automated and streamlined setups.

There are multiple examples. One is running everything locally and one where you have locally only the traditional **resolver** for the clients and the actual ODoH target and relay runs on servers in the cloud.

* [Local development setup with existing certificates](files/local_development_certs_static.md)
* [Local development setup with automated certificates](files/local_development_certs_auto.md)
* [Cloud server setup with existing certificates](files/public_cloud_server_certs_static.md)
* [Cloud server setup with automated certificates](files/public_cloud_server_certs_auto.md)
* [ODoH client](files/odoh_client.md)

## Diagram

Here is a diagram about the setup for better visualization.

![ODoH setup](odoh_cloud_setup.png)