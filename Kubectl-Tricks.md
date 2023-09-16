#tech-tip 
# Kubectl

## Get Pod and Service Cidr

### Pod:
```bash
kubectl cluster-info dump | grep podCIDR | head -1
```


### Service:

```bash
echo '{"apiVersion":"v1","kind":"Service","metadata":{"name":"tst"},"spec":{"clusterIP":"1.1.1.1","ports":[{"port":443}]}}' | kubectl apply -f - 2>&1 | sed 's/.*valid IPs is //'
```

