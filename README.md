
<p align="center"><img src="https://octoeverywhere.com/img/logo.png" alt="OctoEverywhere's Logo" style="width:100px" /></p>
<h1 align="center" style="margin-bottom:20px"><a href="https://octoeverywhere.com/?source=github_api_docs">OctoEverywhere</a></h1>

# OctoEverywhere API Documentation

This git repo contains the source for the OctoEverywhere API doc site:

https://docs.octoeverywhere.com

If you need awesome cloud tools for your 3D printer, checkout [OctoEverywhere.com!](https://octoeverywehre.com/source=github_api_docs)

## Local Dev Setup

- Run Docker
- Open this repo in VS Code
- Click the pop-up to run in a dev container.
- Open http://localhost:8000

The dev container will start, pull the PY dependencies, build, and start a web server on :8000. When you browser connect it will get realtime updates as you edit the markdown.

If you kill the server and need to restart it, run the following command in the dev container terminal:

`mkdocs serve -a 0.0.0.0:8000`
