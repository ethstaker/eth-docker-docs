---
title: "Additional security: Secure Web Proxy"
sidebar_position: 10
sidebar_label: Secure Web Proxy
---

You can use the "[traefik](https://traefik.io/)" secure web proxy to get to the Grafana Dashboard and Prysm Web UI via
https:// instead of insecure http://. It can also be used to encrypt the RPC and WS ports of your execution client, so
they are reachable via https:// and wss:// respectively. In addition, it can be used to separate the consensus client
and validator client to different machines.

You will require a domain name for this to work. Where you buy it is up to you.

As a 450m overview, traefik will be reachable via port 443 / https from the Internet (configurable, could be 8443 if
you prefer). All browsing attempts to it will be checked by traefik for their hostname, and it steers traffic to the
right container thereby: To Grafana, to Prysm Web UI, and to the execution client. Grafana, Prysm web UI, Siren,
RPC and WS ports and for `cl-only.yml` REST ports will be reachable on their configured hostname if traefik
is configured. If you want Grafana and not RPC, for example, simply do not create a DNS (CNAME) entry for RPC.

For example, say I have a domain `example.com`, left the `_HOST` and port settings in `.env` at default, and am running
Prysm with Grafana and Web UI.  `https://grafana.example.com/` will get me to my Grafana dashboard, and
`https://prysm.example.com` to my Prysm Web UI.

## Cloudflare for DNS management

With this option, CloudFlare will provide DNS management as well as DDoS protection. Traefik uses CloudFlare to issue a
Let's Encrypt certificate for your domain. This also automatically updates the IP address of the domain, which is
useful if you are on a dynamic address, such as domestic Internet. This only works for a subdomain such as
`grafana.example.com`, not for the domain itself like `example.com`.

You'll add `traefik-cf.yml` to your `COMPOSE_FILE` in `.env`, for example:
`lighthouse.yml:geth.yml:grafana.yml:traefik-cf.yml`

Create a (free) CloudFlare account and set up your domain, which will require pointing nameservers for your domain to
Cloudflare's servers. How this is done depends on your domain registrar.

You will need a "scoped API token" from CloudFlare's [API page](https://dash.cloudflare.com/profile/api-tokens). Create
a token with `Zone.DNS:Edit`, `Zone.Zone:Read` and `Zone.Zone Settings:Read` permissions, for all zones. Make a note of
the token secret, it will only be shown to you once.

If you want to be [more specific](https://go-acme.github.io/lego/dns/cloudflare/), you can create two scoped API
tokens: One with `Zone.DNS:Edit` for just the domain you wish to manage, and one with `Zone.Zone:Read` and
`Zone.Zone Settings:Read` for all zones.

With that, in the `.env` file:
- Set `DOMAIN` to your domain.
- Set `ACME_EMAIL` to the email address Let's Encrypt will use to communicate with you.
- Set `CF_ZONE_ID` to the Zone ID of the domain, visible in the Overview page of your domain, on the right-hand side
- Set `CF_DNS_API_TOKEN` to the API token with `Edit` rights you just created
under "API".
- Optionally set `CF_ZONE_API_TOKEN` to the API token with `Read` rights, only if you created split permissions.
- Set `DDNS_SUBDOMAIN` to the specific A/AAAA record you want to see created. If you want to update the domain
itself, make this @.
- Set `DDNS_PROXY` to `false` if you do not want CloudFlare to proxy traffic to the subdomain

### CNAMEs and proxy settings

You need CNAMEs or A records for the services you make available. Assuming you have set the subdomain `grafana` with
the IP address of your host, and keeping the default names in `.env`, set the CNAMEs for only the services you use:

- `grafana` is automatically created, proxied, for the Grafana dashboard
- `prysm` CNAME to `grafana.example.com`, proxied, for the Prysm Web UI
- `el` CNAME to `grafana.example.com`, DNS only, for the execution client RPC https:// port
- `elws` CNAME to `grafana.example.com`, DNS only, for the execution client WS wss:// port

If you are using CloudFlare to proxy Grafana / Prysm web, you'll also want to set these:

- SSL/TLS, Overview: "Full" or "Full (strict)" encryption mode
- SSL/TLS, Edge Certificates: Always use HTTPS on, Minimum TLS version to 1.2, Opportunistic Encryption on, TLS 1.3 on,
Automatic HTTPS Rewrites on, Certificate Transparency Monitoring on

## AWS for DNS management

With this option, AWS Route53 will provide DNS management, there is no DDoS protection built in. Traefik uses
Route53 to issue a Let's Encrypt certificate for your domain. It does not create an A record for you, that is left
up to you.

You'll add `traefik-aws.yml` to your `COMPOSE_FILE` in `.env`, for example:
`lighthouse.yml:geth.yml:grafana.yml:traefik-aws.yml`

This setup assumes that you already have an [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
named user profile in `~/.aws` on the node itself. If not, [please create one](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html).
Be sure to specify a default region during `aws configure`. This is not optional.

The IAM user will need to have the AWS-managed `AmazonRoute53ReadOnlyAcces`, `AmazonRoute53AutoNamingFullAccess` and
`AmazonRoute53DomainsFullAccess` policies [attached to it](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html).

With that, in the `.env` file:
- Set `DOMAIN` to your domain.
- Set `ACME_EMAIL` to the email address Let's Encrypt will use to communicate with you.
- Set `AWS_PROFILE` to the profile you want to use. This is the profile name as shown in `~/.aws/config` and
`~/.aws/credentials`, e.g. `default` or whichever name you gave it, *not* the access key id. The profile
**must** contain a region.
- Set `AWS_HOSTED_ZONE_ID` to the Route53 zone you are going to use

### A records and CNAMEs

Assuming you use the default names in `.env`:

- An A record for your first service such as `grafana.example.com`, or on the domain itself `example.com` to use for
CNAMEs. The A record will be the IP address of your node
- Optionally, additional CNAMEs for `grafana`, `prysm`, `el` and `elws`, depending on which services you want to
reverse-proxy on the node

## Traefik common settings

Optionally, you can change the names that services are reachable under, and adjust CNAMEs to match. These are the
`_HOST` variables.

## Separating consensus client and validator client

Eth Docker supports separating the consensus client and validator client on different machines, with TLS encryption
between them.

### Consensus client machine

On the machine that runs the consensus client, you'll need `CLIENT-cl-only.yml` with `CLIENT` one of `teku`,
`lighthouse`, `nimbus`, `lodestar` or `prysm`, as well as one of the `traefik-XXX.yml` files. For example, with
Lighthouse and CloudFlare: `COMPOSE_FILE=lighthouse-cl-only.yml:traefik-cf.yml`.

Traefik needs to be configured as per the above. Make sure you have a DNS entry for the machine, something like
`cl.example.com` if `CL_HOST` is at default and your `DOMAIN` is `example.com`. If you use CloudFlare, you can proxy
this entry.

Make sure port 443/tcp is reachable from the outside. Note this is the CL REST port even for Prysm, what Prysm calls
the "grpc-gateway". The Prysm GRPC port 4000 is not available externally.

In both cases it is prudent to restrict communications to just the IP address of the validator client machine.

### Validator client machine

On the machine that runs the validator client, you'll need `CLIENT-vc-only.yml` with `CLIENT` one of `teku`,
`lighthouse`, `nimbus` or `lodestar`. For example, for Lighthouse: `COMPOSE_FILE=lighthouse-vc-only.yml`

VCs should be interoperable with any CL. Teku and Lighthouse teams test this mutually; for other combinations you'll
want to do some testing yourself.

The `CL_NODE` variable needs to be set to point to the consensus client.

For Teku and Lighthouse: `CL_NODE=https://cl.example.com` , assuming you left the `CL_HOST` variable at default on the
consensus client, the Traefik port at default, and your domain is `example.com`.  
Lighthouse and Teku also support failover nodes, which means you could configure
`CL_NODE=https://cl.example.com,https://cl2.example.com`

