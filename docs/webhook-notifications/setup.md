# Webhook Notifications Setup

There are two ways to receive HTTP POST webhook notifications:

- LAN Webhook Requests
- Public Internet Webhook Requests

### LAN Webhooks

In LAN mode, OctoEverywhere sends webhook HTTP requests from the OctoEverywhere plugin on your local network. This allows you to run a service on your local network that is **not exposed to the public internet.** This can make setting up an HTTP endpoint much easier.

The URL for LAN HTTP requests can be any valid LAN address, such as an IP address like `192.168.1.10`, a local LAN domain like `octopi.local`, or a Docker network bridge like `notificationHandler`.

### Public Internet Webhooks

In this mode, webhook HTTP requests come from the OctoEverywhere service. The HTTP request can be sent to any HTTP endpoint publicly accessible on the internet.

If you're using internet-based webhooks, **we recommend using HTTPS for your webhook URL** to keep the connection secure.

## Optional Secret Key

When you set up your webhook in OctoEverywhere, you can optionally specify any string up to 128 characters that will be sent in every webhook request.

This allows your HTTP server to verify that the request came from OctoEverywhere, since only you and OctoEverywhere know the secret key.

## Create Your Webhook

Now that you have picked how you will receive webhook requests, set up the webhook in OctoEverywhere.

- [Set up a 3D printer](https://octoeverywhere.com/getstarted?source=docs_webhook) with OctoEverywhere
- Go to the [Webhook Notification Setup Page](https://octoeverywhere.com/notifications?handler=webhook&source=docs_webhook).
    - Enter your LAN or internet webhook URL
    - Optionally enter your secret key
- Done 🥳

