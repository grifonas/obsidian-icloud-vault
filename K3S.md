# K3S Tips

## Install

**Add Host to Certificate**:
```bash
curl -sfL [https://get.k3s.io](https://get.k3s.io/) | INSTALL_K3S_EXEC="server --disable=traefik --tls-san k8s.gautzsch.ctmo.io" sh
```
That ^ will add the host `k8s.gautzsch.ctmo.io`  to the generated API cert so that you don;t have to use -insecure when kubectl-ing from outside the host machine.

## Change Main Node IP
After pi restarts the main node might pick up a wrong IP (id WiFi is enabled). 
To change it back:
```bash
kubectl delete node pi4

sudo service k3s restart
```