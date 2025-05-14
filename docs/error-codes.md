# OctoEverywhere Custom Error Codes

OctoEverywhere needs to express connection errors, but it can't use HTTP status codes that might overlap what OctoPrint/Moonraker/etc are using. Thus, we define custom set of error codes to indicate errors from OctoEverywhere.

!!! note
    **These error codes are common to all OctoEverywhere APIs;** including remote access requests, remote access websockets, OctoEverywhere APIs, etc.

## Error Codes

- **600** - **Unknown Error / Server Error**
    - *Temporary* - Something went wrong, try again later.
- **601** - **Printer is Not Connected To OctoEverywhere**
    - *Temporary* - Indicates the OE plugin isn't currently connected to the OE service.
- **602** - **OctoEverywhere's Connection to The Plugin Timed Out.**
    - *Temporary* - The OE plugin is connected, but it took too long to respond to this request.
- **603** - **App Connection Not Found**
    - *Permanent* - The App Connection subdomain used didn't map to a valid App Connection.
- **604** - **App Connection Revoked/Expired**
    - *Permanent* - The App Connection can no longer be used.
        - The App Connection was revoked by the user.
        - The App Connection has expired.
        - The owner's removed the printer from their account.
        - The owner's account was deleted.
- **605** - **App Connection Owner's Account Is No Longer A Supporter.**
    - *Temporary* - The owner account is no longer an OctoEverywhere supporter. If the account is upgraded to supporter status again, this App Connection will continue working as-is.
- **606** - **Invalid App Connection Credentials**
    - *Permanent* - The App Connection requires basic http auth or a Bearer token. The credentials were either missing or incorrect.
- **607** - **File Download Limit Exceeded**
    - *Permanent* - The HTTP **response** body was exceeded the max size for this user's account. The per file limit for this user can be found using the [App Connection Info API](https://octoeverywhere.stoplight.io/docs/octoeverywhere-api-docs/5b0f8eae68257-app-connection-octo-everywhere-api)
- **608** - **File Upload Limit Exceeded**
    - *Permanent* - The HTTP **request** body was exceeded the max size for this user's account. The per file limit for this user can be found using the [App Connection Info API](https://octoeverywhere.stoplight.io/docs/octoeverywhere-api-docs/5b0f8eae68257-app-connection-octo-everywhere-api)
- **609** - **Webcam Back to Back Limit Exceeded**
    - *Temporary* - The user has viewed the webcam too many times per a time window. This time limits can be found in the App OctoEverywhere API.
- **610** - **Plugin Update Required**
    - *(uncommon) Temporary* - The user must update their plugin for this API to work.
- **611** - **No Beta Access**
     - *(uncommon) Temporary* - This API requires beta access that the user doesn't have.
- **612** - **Plugin Error**
     - *(uncommon) Temporary* - There was an error in the plugin that prevented this service call from being successful. Usually caused the a plugin bug, this isn't expected to happen often. If a user hits this constantly, they should contact OctoEverywhere support.
- **613** - **No Active Print**
     - *(uncommon) Temporary* - Currently only used by the [Live Links creation API](https://octoeverywhere.stoplight.io/docs/octoeverywhere-api-docs/632a7f5513f9c-create-live-link-for-app-connections) when the `EndWhenCurrentPrintIsComplete` flag is passed. The `EndWhenCurrentPrintIsComplete` requires that there be a print running on the printer, and thus this error code is returned if there is no print running. In the future, this error code could be used by other APIs, but the API doc for each API would show it.