---
title: Security Audit
sidebar_position: 6
sidebar_label: Security Audit
---

## Reaudit Findings October 2024

Sigma Prime conducted a security audit of the changes to Eth Docker v2.12.3.0 since v2.3, with findings presented in October 2024.

A huge thank-you to both Sigma Prime for the audit, and Ethstaker for funding it.

[Findings as PDF](../../static/pdf/Sigma_Prime_Eth_Docker_Update_2_v2_0.pdf)

There is one informational finding.

### Response

Sigma Prime point out that "Sensitive Data Can Be Handled By Secrets", such as the JWT Secret, which secures the Engine API connection.

Their testing team concurs with the response:
> [Docker secrets] are of marginal utility, [since] if the host is breached on the user running Eth Docker, then by virtue of needing to be
able to run Docker, that user can access the JWT secret no matter how it is stored. In addition, the engine port is
kept within the Docker bridge network by default, not mapped to host.

More to the point, though, Docker does not support secrets unless run in Docker swarm mode.

## Initial Audit Findings May 2023

Sigma Prime conducted a security audit of Eth Docker 2.2.8.4 during March and April 2023, with findings presented on April 30th 2023.

A huge thank-you to both Sigma Prime for the audit, and Ethereum Foundation for funding it.

[Findings as PDF](../../static/pdf/Sigma_Prime_Security_Audit_Findings_2023-05-04_v2.0.pdf)

There are one medium severity and four informational findings. The medium-severity finding is about the entropy used for JWT secret,
API manager token in Nimbus and Lodestar, Prysm wallet password, and Teku cert password: Entropy comes from `$RANDOM` and is therefore only 16 bits.

### Response

Eth Docker v2.3 addresses these findings. It now uses 64 bits of entropy and SHA-256 hash.

- Secrets entropy

Users that expose the Engine API with `ee-shared.yml` or `ee-traefik.yml` need to make sure that it is firewalled to trusted IPs.

Users that did so before Eth Docker v2.3 may want to, in addition, update with `./ethd update`, stop the stack with `./ethd stop`,
delete the `jwtsecret` docker volume with `docker volume ls` and `docker volume rm`, and start the stack with `./ethd up`. 

- ED-05 recommends: "Use of clipboard for sensitive strings. Consider giving the user the option of using the clipboard for sensitive
strings instead of storing the data in a file."

This was not addressed. Engine API JWT secret and keymanager API token need to be present in files for the clients to function. Prysm wallet password is stored
in a file so the Prysm client can open it. Keymanager API token and Prysm wallet password are printed to stdout if the user requests them.
