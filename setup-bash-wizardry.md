# Some useful commands in the setup scripts

## Mainly to get things set up properly or set variables.

### Check if a deployment exists for track logs
```bash
kubectl get deployment deployment-name -n namespace &> /dev/null && echo "deployment-name does exist"
```

### Wait for pod(s) to be ready
```bash
while [ "$(kubectl get pods -n namespace -l=app='value' -o jsonpath='{.items[*].status.containerStatuses[0].ready}')" != "true" ]; do
   sleep 5
   echo "Waiting for pods to be ready."
done
```


### Get the pod name programmatically
```bash
agent variable set POD_NAME $(kubectl get pods -n namespace -o=jsonpath='{.items[?(@.metadata.labels.app=="value")].metadata.name}')
```

### Get the pod/service IP programmatically
```bash
agent variable set POD_IP $(kubectl get pods -n namespace -o=jsonpath='{.items[?(@.metadata.labels.app=="value")].status.podIP}')
agent variable set CLUSTERIP $(kubectl get svc name -n namespace -o=jsonpath='{.spec.clusterIP}')
agent variable set INGRESS_IP $(kubectl get svc name -n namespace -o=jsonpath='{.status.loadBalancer.ingress.*.ip}')
```

### Generate IP blocks from ipdeny to go into a network set and format them for YAML (example is Cuba)
```bash
curl --silent https://www.ipdeny.com/ipblocks/data/aggregated/cu-aggregated.zone >> cu-ip-list.txt
sed 's/^/    -  /' < cu-ip-list.txt >> cu-ip-list-edit.txt
CU_IPS=$(cat cu-ip-list-edit.txt)
```
```yaml
cat > cuba-nws.yaml << EOF
apiVersion: projectcalico.org/v3
kind: GlobalNetworkSet
metadata:
  name: embargoed-countries-cuba
  labels:
    country: "cuba"
spec:
  nets:
 # These IP address(es) are known IP blocks from this country
$CU_IPS
EOF
```

### Write something to a file

```yaml
cat > something.yaml <<EOF
yaml: goesHere
EOF
```


### Generate a list of IPs from a block (see above) and pick one to set as a variable for testing inside the workshop
prips will be able to generate IP addresses from a block
```bash
apt install prips
CU_LINE=$(awk 'END { print NR }' cu-ip-list.txt)
bash -c 'CU_LINE_RAND=$((1 + $RANDOM % $CU_LINE)) && echo'
CU_LINE_RAND=$(($RANDOM % $CU_LINE))
# use the random number to sample a random ipblock from each country
CU_BLOCK=$(sed -n "$CU_LINE_RAND"p cu-ip-list.txt)
ONE_CU_IP=$(prips $CU_BLOCK | sed -n $((1 + $RANDOM % $(prips $CU_BLOCK | awk 'END { print NR }')))p)
agent variable set ONE_CU_IP "$ONE_CU_IP"
```



## Calico specific set up

### Enable intrusion detection
```yaml
kubectl create -f -<<EOF
apiVersion: operator.tigera.io/v1
kind: IntrusionDetection
metadata:
  name: tigera-secure
EOF
```

### Attempt to get Calico UI settings (Service Graph) and hide the Tigera infrastructure from the default view
```bash
kubectl get uisettings --no-headers -o custom-columns=":metadata.name" | while read line; do
    echo "Text read from file: $line"
    kubectl patch uisettings $line -p '{"spec": {"view": {"nodes": [{"id": "layer/cluster-settings.layer.tigera-infrastructure", "name": "cluster-settings.layer.tigera-infrastructure", "type": "layer", "hide": true}]}}}'
done
```

### Enable Policy Recommendation
```yaml
kubectl create -f -<<EOF
apiVersion: operator.tigera.io/v1
kind: PolicyRecommendation
metadata:
  name: tigera-secure
EOF
kubectl apply -f -<<EOF
apiVersion: projectcalico.org/v3
kind: PolicyRecommendationScope
metadata:
  name: default
spec:
  interval: 1m0s
  namespaceSpec:
    recStatus: Enabled
    selector: '!(projectcalico.org/name starts with ''tigera-'') && !(projectcalico.org/name
      starts with ''calico-'') && !(projectcalico.org/name starts with ''kube-'')'
  stabilizationPeriod: 2m0s
status: {}
EOF
```

### Calico Enterprise login
```bash
agent variable set UI_SECRET $(cat /root/ui-secret.txt)
```

### Enable L7 log collection

```yaml
kubectl apply -f -<<EOF
apiVersion: operator.tigera.io/v1
kind: ApplicationLayer
metadata:
  name: tigera-secure
spec:
  logCollection:
    collectLogs: Enabled
    logIntervalSeconds: 5
    logRequestsPerInterval: -1
EOF
```