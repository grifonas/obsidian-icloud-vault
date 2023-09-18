#cbre 
# Key Pair Authentication in Snowflake

Taken from [here](https://docs.snowflake.com/en/user-guide/key-pair-auth).

1. Generate private key:
```bash
openssl genrsa 2048 | openssl pkcs8 -topk8 -inform PEM -out rsa_key.p8 -nocrypt
```
2. Public:
```bash
openssl rsa -in rsa_key.p8 -pubout -out rsa_key.pub
```
3. Assign your key to the SNowflake user:
```SQL
ALTER USER contiamo SET RSA_PUBLIC_KEY='MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA1M4Nu+gpLToXFFYHRqL9
rKILR9BUx1PkYB//9ATRH/xh55F0A3gpzaYNEtkaQhjaFzl3FJyyEnbwDdRmcl0K
PO0vXLrWuNw7XZB92FxXmDLN79rdANDaS0pAXoySeW1VpQv9Hvr6IUEzZh3+eL2S
7WqkcduuW/bB/H9NdcY7XrB9m7uOHGJ4zUhPV10lxC7hdor2v5wko4e1DGj9wMul
P3mV+KKLT+kko4fFU/R+r5uatNChFI0kuV4Jkfl36BmTlAuornvacdeVQ5Yb2iVn
st+p45vlXM4lLdr0B28qZVS629VQTOaug17y+zLxZ3OoiVpPnNWwhzUTZEdKUuDg
VQIDAQAB';
```

**Note the lack of " ---BEGIN... and --- END"**


terraform/pipelines
cbre-gdp-replatforming-sandbox-pipelines