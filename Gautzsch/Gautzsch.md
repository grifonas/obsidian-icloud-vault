# Gautzsch Setup

## Connection:
- We connect to their network using a Windows machine running in our AWS Details in [1Password](https://start.1password.com/open/i?a=7BGSQM4UKVEJTK5H3MNEJ67YL4&v=vjr4zmjabwzxt2b5j5uc3r33oq&i=ht2ypv64dmplemivukvqixwp3e&h=contiamo.1password.com)
- One of us needs to connect to the Windows machine and from there to their Palo Alto VPN
- Once **ANY ONE OF US** is connected, EVERYONE can connect to the network us9ng outr Tailnet. As long as you're connected to the Tailsnet AND you have `--accept-routes` the network is accessible.

## Jupyter Hub

- Installed using Terraform. Code [here](https://github.com/contiamo/project-ops/tree/main/Gautzsch/stock-anomalies)
- Host: http://jupyterhub.gautzsch.ctmo.io

### Jupyter Hub Image
Lived in https://github.com/contiamo/jupyter-hub
If you need to add things, use that repo. Merge, approve the release PR, wait for it to get built.