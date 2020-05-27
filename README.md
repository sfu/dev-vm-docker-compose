# dev-vm-docker-compose

A simple Docker Compose configuration that orchestrates setting up the tools necessary to use make applications running on a development VM availavle through a public hostname with HTTPS provided by Let's Encrypt.

## Usage

- Copy the docker-comopse.local.yml.example file to docker-compose.local.yml
- Copy the .env.example file to .env
- Edit docker-compose.local.yml and set the `mousehouse` environment variables as follows:
  - `MM_SERVER`: mm.its.sfu.ca
  - `MM_USERNAME`: a Men & Mice user who has the ability to manage your target DNS zone via the SOAP interface. For LCS, this is `LCP-SOAP`.
  - `MM_PASSWORD`: password for `MM_USERNAME`. For LCS, see the entry for the `LCP-SOAP` account in KeePass.
  - `CNAME_TO`: the FQDN of your NSX Load Balancer. This _must_ end with a trailing period (e.g. `kipling.lcs-dev.its.sfu.ca.`)
  - `DNS_ZONE_REF`: the internal Men & Mice reference to the DNS zone where CNAME entries will be created. For LCS, consult the [LCS DNS Zones](https://confluence.its.sfu.ca/x/3p2PAw) page in Confluence for the appropriate entry.
- Start the compose stack in the background: `docker-compose up -d`
- Start a container or compose stack for your application, passing the `VIRTUAL_HOST` and `LETSENCRYPT_HOST` environment varibles with your desired hostname. For example:

```
docker run --rm \
  -e "VIRTUAL_HOST=grahamb-nginx.lcs-dev.its.sfu.ca" \
  -e "LETSENCRYPT_HOST=grahamb-nginx.lcs-dev.its.sfu.ca" \
  -d --name nginx-test \
  nginx:latest
```

This will:

- Configure the nginx-proxy container to forward requests on port 80 and 443 to your backend container
- Generate a certificate using Let's Encrypt and configure the nginx-proxy container to use it
- Create a DNS CNAME from grahamb-nginx.lcs-dev.its.sfu.ca to the value of the `CNAME_TO` environment variable you specified

When the container is stopped (`e.g. docker stop nginx-test`), the revers will happen; the proxy configuration is removed and the CNAME is deleted.
