
1. Install the ECK operator and CRDs:
```bash
kubectl apply -f https://download.elastic.co/downloads/eck/2.14.0/crds.yaml
kubectl apply -f https://download.elastic.co/downloads/eck/2.14.0/operator.yaml
```
2. Create the deployment namespaces
```bash
cd yaml-manifests
kubectl apply -f all-namespaces/namespaces/namespaces.yml
```

3. useful helm commands
```bash
helm repo add traefik https://traefik.github.io/charts
helm repo list  
helm search repo traefik 
helm repo update
```

4. install Traefik with helm
```bash
helm upgrade -i traefik traefik/traefik --namespace=traefik-v2

kubectl get services -n traefik-v2 -w  # copy the `EXTERNAL-IP` that you see because you will need to use it in Elastic service
```

## Deploy the monitoring cluster

5. Create the s3 secret for the monitoring cluster snapshots repo
```bash
kubectl create secret -n elastic-mon generic s3-secret-settings \
--from-literal=s3.client.secondary.access_key=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT \
--from-literal=s3.client.secondary.secret_key=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT
```
6. Create the kibana saved objects encryption secret
```bash
kubectl create secret -n elastic-mon generic kibana-secret-settings \
   --from-literal=xpack.security.encryptionKey=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT \
   --from-literal=xpack.reporting.encryptionKey=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT \
   --from-literal=xpack.encryptedSavedObjects.encryptionKey=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT
```

7. Deploy and expose the monitoring cluster via the ingress controller
```bash
kubectl apply -f monitoring-cluster/elasticsearch/elasticsearch.yml
kubectl apply -f monitoring-cluster/kibana/kibana.yml
kubectl apply -f monitoring-cluster/traefik/traefik-server-transport.yml
kubectl apply -f monitoring-cluster/traefik/traefik-redirect-to-https-middleware.yml
# update the IP in the URL with the Traefik service IP
kubectl apply -f monitoring-cluster/traefik/traefik-ingress-route-elasticsearch-websecure.yml
kubectl apply -f monitoring-cluster/traefik/traefik-ingress-route-kibana-websecure.yml
```
8. Retrieve the `elastic` user password
```bash
kubectl get secret elasticsearch-mon-es-elastic-user -n elastic-mon -o go-template='{{.data.elastic | base64decode }}' 
```

## Deploy the production cluster

9. Create the s3 secret
```bash
kubectl create secret -n elastic-prod generic s3-secret-settings \
--from-literal=s3.client.secondary.access_key=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT \
--from-literal=s3.client.secondary.secret_key=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT
```

1. Create the kibana saved objects encryption secret
```bash
kubectl create secret -n elastic-prod generic kibana-secret-settings \
   --from-literal=xpack.security.encryptionKey=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT \
   --from-literal=xpack.reporting.encryptionKey=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT \
   --from-literal=xpack.encryptedSavedObjects.encryptionKey=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT
```

7. Deploy and expose the monitoring cluster via the ingress controller
```bash
kubectl apply -f production-cluster/elasticsearch/elasticsearch.yml
kubectl apply -f production-cluster/kibana/kibana.yml
kubectl apply -f production-cluster/fleet/fleet-server.yml
kubectl apply -f production-cluster/traefik/traefik-server-transport.yml
kubectl apply -f production-cluster/traefik/traefik-redirect-to-https-middleware.yml
kubectl apply -f production-cluster/traefik/traefik-ingress-route-elasticsearch-websecure.yml
kubectl apply -f production-cluster/traefik/traefik-ingress-route-kibana-websecure.yml

kubectl apply -f production-cluster/fleet/fleet-server.yml
kubectl apply -f fleet/traefik-ingress-route-fleet-websecure.yml
```
```
8. Retrieve the `elastic` user password
```bash
kubectl get secret elasticsearch-mon-es-elastic-user -n elastic-mon -o go-template='{{.data.elastic | base64decode }}' 
```

1. - - - Hint: watch namespace events and specific container logs - - - 

```bash
kubectl get events --sort-by='.metadata.creationTimestamp' -n elastic-prod -w
```

8. Retrieve the Elasticsearch config an d user password: 
```bash
kubectl get secret elasticsearch-prod-es-data-es-config -n elastic-prod -o go-template='{{index .data "elasticsearch.yml" | base64decode}}'
kubectl get secret elasticsearch-prod-es-elastic-user -n elastic-prod -o go-template='{{.data.elastic | base64decode }}' 
```



1.  Check this in the browser URL: 
```
demo.fleet.<ip>.nip.io
```