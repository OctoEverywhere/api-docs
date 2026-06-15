---
title: OctoPrint API Key Generation
description: Ask the App Connection Portal to generate an OctoPrint API key so your app can securely authenticate to a user's OctoPrint instance.
og_title: Generate OctoPrint API Keys During App Setup
og_description: Let OctoEverywhere help your app collect the OctoPrint API key it needs as part of the App Connection portal flow.
authors:
    - Quinn Damerell
date: 2026-06-15
---

# OctoPrint API Key Generation

The App Connection Portal can optionally generate an OctoPrint API key for your app. This makes OctoPrint setup easier because users do not need to find and paste an API key manually.

## Enable API Key Generation

Add `octoPrintApiKeyAppName` to the portal start URL.

```text
https://octoeverywhere.com/appportal/v1/?appid=<your_app_id>&octoPrintApiKeyAppName=<url_encoded_app_name>
```

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `octoPrintApiKeyAppName` | string | No | URL-encoded app name shown in OctoPrint's API key settings page. The app name must be between 2 and 60 characters. If present, the portal attempts to create an OctoPrint API key. |

When API key generation succeeds, the portal includes `octoPrintApiKey` on the completion URL.

## Completion Parameter

| Name | Type | Description |
| ---- | :--: | ----------- |
| `octoPrintApiKey` | string | The generated OctoPrint API key. This parameter is only present when generation was requested and succeeded. |

If `octoPrintApiKey` is present, the key is ready to use. If it is missing, your app should continue setup and ask the user to provide an OctoPrint API key or legacy authentication token manually.

## Why Key Generation Can Fail

OctoPrint API key generation is best effort. It can fail for several reasons:

- The OctoPrint server was offline or not connected to OctoEverywhere.
- The installed OctoEverywhere plugin version was too old.
- The OctoPrint plugin did not have auth available.
- The user failed the email-based access challenge.
- Another printer, plugin, account, or connection condition prevented key creation.

Historically, API key generation succeeded for roughly 95% of attempts, but your app must handle the missing-key case.

## How It Works Securely

This section is background information. You do not need to implement this security flow yourself; the portal and plugin handle it.

The OctoEverywhere plugin first performs an RSA challenge against the OctoEverywhere service to verify that the service is authentic. The plugin is already connected over a secure WebSocket with a valid SSL certificate; the RSA challenge adds another layer of trust.

The plugin can generate an API key for itself after installation. That local API key lets the plugin make authenticated OctoPrint calls, but it is stored on the OctoPrint device and is not transmitted to OctoEverywhere.

After the plugin establishes trust, the service can make a special call that asks the plugin to add its locally stored API key to the request, allowing a new app key to be created. This special call can only be made by the OctoEverywhere service.

Because generating an OctoPrint API key grants meaningful access, the portal may challenge the user again. The user must already be logged in to OctoEverywhere, including two-factor auth if configured. If additional verification is required, the portal sends a unique approval email. Only after the user proves access to that email account does the service generate the OctoPrint API key.

If the user cannot complete the email challenge, the portal can still finish the App Connection setup, but `octoPrintApiKey` will not be returned.
