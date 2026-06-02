# OctoEverywhere Custom Error Codes

OctoEverywhere uses a common set of unique HTTP status codes to express errors without overlapping with codes returned by proxied services.

!!! tip
    **These error codes are common to all OctoEverywhere APIs,** including remote access requests, remote access WebSockets, and OctoEverywhere APIs.

The following list explains each error code, what it means, and whether the condition is temporary or permanent. Existing codes will not change, though new codes may be added over time.

## Error Codes

- **600** - **Unknown Error / Server Error**
    - *Temporary* - Something went wrong. Try again later.
- **601** - **Printer Is Not Connected to OctoEverywhere**
    - *Temporary* - The OE plugin is not currently connected to the OE service.
- **602** - **OctoEverywhere's Connection to the Plugin Timed Out**
    - *Temporary* - The OE plugin is connected, but it took too long to respond to this request.
- **603** - **App Connection Not Found**
    - *Permanent* - The App Connection subdomain did not map to a valid App Connection.
- **604** - **App Connection Revoked/Expired**
    - *Permanent* - The App Connection can no longer be used:
        - The App Connection was revoked by the user.
        - The App Connection has expired.
        - The printer owner removed the printer from their account.
        - The printer owner's account was deleted.
- **605** - **App Connection Owner's Account Is No Longer a Supporter**
    - *Temporary* - The owner account is no longer an OctoEverywhere supporter. If the account is upgraded to supporter status again, this App Connection will continue working as-is.
- **606** - **Invalid App Connection Credentials**
    - *Permanent* - The App Connection requires basic HTTP auth or a Bearer token. The credentials were missing or incorrect.
- **607** - **File Download Limit Exceeded**
    - *Permanent* - The HTTP **response** body exceeded the maximum size for this user's account. The per-file limit can be found using the [App Connection Info API](https://octoeverywhere.stoplight.io/docs/octoeverywhere-api-docs/5b0f8eae68257-app-connection-octo-everywhere-api).
- **608** - **File Upload Limit Exceeded**
    - *Permanent* - The HTTP **request** body exceeded the maximum size for this user's account. The per-file limit can be found using the [App Connection Info API](https://octoeverywhere.stoplight.io/docs/octoeverywhere-api-docs/5b0f8eae68257-app-connection-octo-everywhere-api).
- **609** - **Webcam Back to Back Limit Exceeded**
    - *Temporary* - The user has viewed the webcam too many times within a time window. The current limits can be found in the App OctoEverywhere API.
- **610** - **Plugin Update Required**
    - *(uncommon) Temporary* - The user must update their plugin for this API to work.
- **611** - **No Beta Access**
     - *(uncommon) Temporary* - This API requires beta access that the user doesn't have.
- **612** - **Plugin Error**
     - *(uncommon) Temporary* - An error in the plugin prevented this service call from completing successfully. Usually caused by a plugin bug, this is not expected to happen often. If a user hits this repeatedly, they should contact OctoEverywhere support.
- **613** - **No Active Print**
     - *(uncommon) Temporary* - Currently used only by the [Live Links creation API](https://octoeverywhere.stoplight.io/docs/octoeverywhere-api-docs/632a7f5513f9c-create-live-link-for-app-connections) when the `EndWhenCurrentPrintIsComplete` flag is passed. That flag requires an active print, so this error code is returned when no print is running. In the future, this error code could be used by other APIs, but each API doc will call it out when applicable.
