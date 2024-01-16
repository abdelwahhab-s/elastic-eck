
1. Install the ECK operator and CRDs:
```bash
kubectl apply -f https://download.elastic.co/downloads/eck/2.10.0/crds.yaml
kubectl apply -f https://download.elastic.co/downloads/eck/2.10.0/operator.yaml
```
2. Create the deployment namespaces
```bash
kubectl apply -f namespaces/namespaces.yml
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
helm install --namespace=traefik-v2 traefik traefik/traefik

kubectl get services -n traefik-v2 -w  # copy the `EXTERNAL-IP` that you see because you will need to use it in Elastic service
```

5. Create the s3 secret
```bash
kubectl create secret -n elastic-prod generic s3-secret-settings \
--from-literal=s3.client.secondary.access_key=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT \
--from-literal=s3.client.secondary.secret_key=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT
```

6. Apply the ES configs
``` bash
kubectl apply -f elasticsearch/elasticsearch.yml
kubectl apply -f elasticsearch/traefik-server-transport.yml
kubectl apply -f elasticsearch/traefik-redirect-to-https-middleware.yml
kubectl apply -f elasticsearch/traefik-ingress-route-elasticsearch-websecure.yml 
```

7. - - - Hint: watch namespace events and specific container logs - - - 

```bash
kubectl get events --sort-by='.metadata.creationTimestamp' -n elastic-prod -w
```

8. Retrieve the Elasticsearch config an d user password: 
```bash
kubectl get secret elasticsearch-prod-es-data-es-config -n elastic-prod -o go-template='{{index .data "elasticsearch.yml" | base64decode}}'
kubectl get secret elasticsearch-prod-es-elastic-user -n elastic-prod -o go-template='{{.data.elastic | base64decode }}' 
```

9. Create Kibana secrets: 
```bash
kubectl create secret -n elastic-prod generic kibana-secret-settings \
   --from-literal=xpack.security.encryptionKey=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT \
   --from-literal=xpack.reporting.encryptionKey=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT \
   --from-literal=xpack.encryptedSavedObjects.encryptionKey=Xv/6tc15Hb+tV+7kCvKoTetFLFVxYcluZfwz9DyV7KT
```

10. Apply the Kibana configs
``` bash
kubectl apply -f kibana/kibana-small.yml
kubectl apply -f kibana/traefik-ingress-route-kibana-websecure.yml 
```

11. Apply the Fleet Server configs
``` bash
kubectl apply -f fleet/fleet-server.yml
kubectl apply -f fleet/traefik-ingress-route-fleet-websecure.yml
```

12. Check this in the browser URL: 
```
demo.fleet.<ip>.nip.io
```