---
title:  Updating the software.
sidebar_position: 4
sidebar_label: Client Updates
---

This project does not monitor client versions. It is up to you to decide that you
are going to update a component. When you are ready to do so, the below instructions
show you how to.

You can find the current version of your client by running `./ethd version`, assuming the
node is up and running

## Automated update

Inside the project directory, run:
  `./ethd update`

This will update Eth Docker, all Ethereum clients, and migrate your `.env` settings over to a fresh copy
from `default.env`.

If you want to reset your client version targets, run `./ethd update --refresh-targets` instead.

Restart changed containers with `./ethd up`.

## Optional: Manually update Eth Docker

Inside the project directory, run:
  `git pull`

Then `cp .env .env.bak` and `cp default.env .env`, and set variables inside `.env`
the way you need them, with `.env.bak` as a guide.

Updating the tool itself is not always necessary. Please refer to the [Changelog](../About/Changelog.md) to see
whether changes have been made that you may want to use.

## Optional: Manually update the client "stack"

If you are using binary build files - the default - you can update everything
in the client "stack" with `./ethd cmd build --pull`. If you
run shared components in a different directory, such as the execution client,
you'd `cd` into those directories and run the command there.

And restart changed containers: `./ethd up`

Then verify that the components are coming up okay again by looking at logs:
- `./ethd logs -f consensus` for the consensus client
- `./ethd logs -f validator` for the validator client
- `./ethd logs -f execution` for the execution client
