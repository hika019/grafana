
# Create Token

```bash
TOKEN="apiv3_$(openssl rand -base64 24)"

cat > admin-token.json <<EOF
{"token":"$TOKEN","name":"_admin"}EOF

kubectl create secret generic influxdb-admin-token \
  -n grafana \
  --from-file=admin-token.json


```
