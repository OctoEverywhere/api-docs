# Webhook Notifications Setup

There are two ways you can receive HTTP POST webhook notifications:

- LAN Webhook Requests
- Public Internet Webhook Requests

### LAN Webhooks

In LAN mode, OctoEverywhere will send webhook http requests from the OctoEverywhere plugin on your local network. This allows you to run a service on your local network that's **not exposed to the public internet.** This can make setting up an HTTP endpoint much easier.

Then URL for LAN HTTP requests can be any valid LAN address. Such as IP addresses `192.168.1.10`, local LAN domains `octopi.local`, Docker network bridges `notificationHandler`, etc.

### Public Internet Webhooks

The webhook HTTP requests will come from the OctoEverywhere service in this mode. The HTTP request can be sent to any HTTP endpoint publicly accessible on the public internet.

If you're using internet-based webhooks, **it's recommended that you use https for your webhook URL** to keep the connection secure.

## Optional - Secret Key

When you set up your webhook in OctoEverywhere, you can optionally specify any string less than or equal to 128 chars that will be sent in every webhook request.

This allows your HTTP server to verify that the request came from OctoEverywhere since only OctoEverywhere and you know the secret key.

## Create Your Webhook

Now that you have picked how you're going to build your webhook, you need to setup the  webhook in OctoEverywhere.

- [Setup a 3D printer](https://octoeverywhere.com/getstarted?source=docs_webhook) with OctoEverywhere
- Go to the [Webhook Notification Setup Page.](https://octoeverywhere.com/notifications?handler=webhook&source=docs_webhook)
    - Enter your LAN or Internet Webhook URL
    - Optionally enter your Secret Key
- Done 🥳

