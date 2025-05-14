# 🌍 3D Printer API Remote Access

Get secure access to your 3D printer's full APIs from anywhere. Allowing you to easily monitor, control, or manage your 3D printers from a service, app, or anything!

API Remote Access works with:

- OctoPrint
- Klipper / Moonraker
- Bambu Lab OS
- Elegoo OS
- Creality OS


Full docs coming soon, but here's the quick version:

- [Setup your 3D printer](https://octoeverywhere.com/getstarted?source=docs) with OctoEverywhere
- Create A [Shared Connection](https://octoeverywhere.com/sharedconnections?source=docs)
- Use any API or Websocket as you would locally, but replace the Shared Connection for the local printer's IP or hostname.

Our relay service supports all HTTP request types (GET, POST, etc), websockets, and a MQTT proxy via websocket for Bambu Lab 3D printers.

