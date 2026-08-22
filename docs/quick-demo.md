kubectl logs -n ingress-nginx deployment/ingress-nginx-controller --tail=200 |
grep -E 'shopnow|static|404|500|502|503'



```bash
aws sts get-caller-identity

aws eks update-kubeconfig \
  --region ap-south-1 \
  --name shopnow-app-eks

aws eks describe-cluster \
  --region ap-south-1 \
  --name shopnow-app-eks \
  --query 'cluster.{Name:name,Status:status,Version:version,Endpoint:endpoint,Created:createdAt}' \
  --output table

kubectl cluster-info
kubectl get nodes -o wide
kubectl get namespaces

kubectl get deployment,statefulset,daemonset,pod,service,ingress \
  -n shopnow-ns \
  -o wide

kubectl get events \
  -n shopnow-ns \
  --sort-by='.lastTimestamp'

kubectl get service \
  -n ingress-nginx \
  ingress-nginx-controller \
  -o wide

kubectl get service \
  -n ingress-nginx \
  ingress-nginx-controller \
  -o jsonpath='Load Balancer: http://{.status.loadBalancer.ingress[0].hostname}{"\n"}ShopNow User: http://{.status.loadBalancer.ingress[0].hostname}/shopnow/{"\n"}ShopNow Admin: http://{.status.loadBalancer.ingress[0].hostname}/shopnow/admin/{"\n"}ShopNow API: http://{.status.loadBalancer.ingress[0].hostname}/shopnow/api/{"\n"}'

curl -IL --max-time 15 "http://$(kubectl get service -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')/shopnow/"

curl -IL --max-time 15 "http://$(kubectl get service -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')/shopnow/admin/"

curl -i --max-time 15 "http://$(kubectl get service -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')/shopnow/api/"

kubectl get ingress \
  -n shopnow-ns \
  -o custom-columns='INGRESS:.metadata.name,PATHS:.spec.rules[*].http.paths[*].path,SERVICE:.spec.rules[*].http.paths[*].backend.service.name,PORT:.spec.rules[*].http.paths[*].backend.service.port.number,ADDRESS:.status.loadBalancer.ingress[*].hostname'

kubectl auth can-i --list -n shopnow-ns
kubectl auth can-i get pods -n shopnow-ns
kubectl auth can-i get pods/log -n shopnow-ns
kubectl auth can-i get services -n shopnow-ns
kubectl auth can-i get ingress -n shopnow-ns
kubectl auth can-i create deployments -n shopnow-ns
kubectl auth can-i delete deployments -n shopnow-ns
kubectl auth can-i '*' '*' --all-namespaces

kubectl get role,rolebinding \
  -n shopnow-ns \
  -o wide

kubectl describe role,rolebinding \
  -n shopnow-ns

kubectl get secretstore,externalsecret \
  -n shopnow-ns

kubectl get secret mongo-secret \
  -n shopnow-ns

kubectl get pods \
  -n monitor-ns \
  -o wide

kubectl get servicemonitor,prometheusrule \
  -n monitor-ns

kubectl top nodes
kubectl top pods -n shopnow-ns

kubectl logs \
  -n shopnow-ns \
  deployment/frontend \
  --tail=100

kubectl logs \
  -n shopnow-ns \
  deployment/admin \
  --tail=100

kubectl logs \
  -n shopnow-ns \
  deployment/backend \
  --tail=100

aws ecr describe-repositories \
  --region ap-south-1 \
  --query 'repositories[?contains(repositoryName, `shopnow-dev`)].{Name:repositoryName,URI:repositoryUri,Created:createdAt}' \
  --output table

aws secretsmanager describe-secret \
  --region ap-south-1 \
  --secret-id shopnow/mongo \
  --query '{Name:Name,ARN:ARN,Updated:LastChangedDate}' \
  --output table
```

-------------------------------------------

Mongo DB:

```bash
kubectl get deployment,pod,service,endpoints,pvc \
  -n shopnow-ns \
  -l app=mongo \
  -o wide

kubectl describe deployment mongo \
  -n shopnow-ns

kubectl describe service mongo \
  -n shopnow-ns

kubectl get endpoints mongo \
  -n shopnow-ns \
  -o yaml

kubectl get pvc \
  -n shopnow-ns \
  -o wide

kubectl get pv \
  -o wide

kubectl get secret mongo-secret \
  -n shopnow-ns \
  -o jsonpath='{.data}' |
  jq 'keys'

kubectl get secret mongo-secret \
  -n shopnow-ns \
  -o jsonpath='{.data.MONGODB_URI}' |
  base64 --decode
echo

aws secretsmanager describe-secret \
  --region ap-south-1 \
  --secret-id shopnow/mongo \
  --query '{Name:Name,ARN:ARN,Created:CreatedDate,Updated:LastChangedDate,RotationEnabled:RotationEnabled}' \
  --output table

aws secretsmanager get-secret-value \
  --region ap-south-1 \
  --secret-id shopnow/mongo \
  --query SecretString \
  --output text |
  jq

kubectl exec \
  -n shopnow-ns \
  deployment/mongo \
  -- mongosh \
  --quiet \
  --eval 'db.adminCommand({ping:1})'

kubectl exec \
  -n shopnow-ns \
  deployment/mongo \
  -- mongosh \
  --quiet \
  --eval 'db.adminCommand({serverStatus:1}).version'

kubectl exec \
  -n shopnow-ns \
  deployment/mongo \
  -- mongosh \
  --quiet \
  --eval 'db.adminCommand({listDatabases:1}).databases'

kubectl exec \
  -n shopnow-ns \
  deployment/mongo \
  -- mongosh \
  --quiet \
  --eval 'db.getMongo().getDBNames().forEach(function(databaseName){const currentDb=db.getSiblingDB(databaseName); print(databaseName); printjson(currentDb.getCollectionNames())})'

kubectl exec \
  -n shopnow-ns \
  deployment/mongo \
  -- mongosh \
  --quiet \
  --eval 'db.getMongo().getDBNames().forEach(function(databaseName){const currentDb=db.getSiblingDB(databaseName); currentDb.getCollectionNames().forEach(function(collectionName){print(databaseName+"."+collectionName+": "+currentDb.getCollection(collectionName).countDocuments({}))})})'

kubectl logs \
  -n shopnow-ns \
  deployment/mongo \
  --tail=200

kubectl logs \
  -n shopnow-ns \
  deployment/mongo \
  --previous \
  --tail=200

kubectl top pod \
  -n shopnow-ns \
  -l app=mongo

kubectl port-forward \
  -n shopnow-ns \
  service/mongo \
  27017:27017
```


kubectl exec -it -n shopnow-ns deployment/mongo -- mongosh

kubectl exec -it -n shopnow-ns deployment/mongo -- sh -c 'exec mongosh --username "$MONGO_INITDB_ROOT_USERNAME" --password "$MONGO_INITDB_ROOT_PASSWORD" --authenticationDatabase admin'

show dbs
use shopnow
show collections
db.getCollectionNames()
db.users.find().limit(10)
db.products.find().limit(10)
db.orders.find().limit(10)
db.users.countDocuments({})
db.products.countDocuments({})
db.orders.countDocuments({})

User:  http://a2d7eee8d8179427fa36d881be68d64a-277526266.ap-south-1.elb.amazonaws.com/shopnow/
Admin: http://a2d7eee8d8179427fa36d881be68d64a-277526266.ap-south-1.elb.amazonaws.com/shopnow/admin/
API:   http://a2d7eee8d8179427fa36d881be68d64a-277526266.ap-south-1.elb.amazonaws.com/shopnow/api/health



kubectl logs -n ingress-nginx deployment/ingress-nginx-controller --tail=200 |
grep -E 'shopnow|static|404|500|502|503'

kubectl logs -n ingress-nginx deployment/ingress-nginx-controller --tail=200 |
grep -E 'shopnow|static|200'


```bash
# Live backend API logs
kubectl logs -f -n shopnow-ns deployment/backend --tail=100
```

```bash
# Live ingress logs showing every API request and HTTP status
kubectl logs -f -n ingress-nginx deployment/ingress-nginx-controller --tail=100 |
grep --line-buffered '/shopnow/api'
```

```bash
# Show only API errors
kubectl logs -f -n ingress-nginx deployment/ingress-nginx-controller --tail=100 |
grep --line-buffered -E '/shopnow/api.* (400|401|403|404|500|502|503) '
```

Keep the log command running, then hit APIs from another WSL terminal:

```bash
curl -i http://a2d7eee8d8179427fa36d881be68d64a-277526266.ap-south-1.elb.amazonaws.com/shopnow/api/health
```

```bash
curl -i http://a2d7eee8d8179427fa36d881be68d64a-277526266.ap-south-1.elb.amazonaws.com/shopnow/api/products
```

```bash
curl -i http://a2d7eee8d8179427fa36d881be68d64a-277526266.ap-south-1.elb.amazonaws.com/shopnow/api/categories
```

Follow backend logs from every backend pod:

```bash
kubectl logs -f -n shopnow-ns \
  -l app=backend \
  --all-containers=true \
  --prefix=true \
  --tail=100 \
  --max-log-requests=10
```

Watch Kubernetes events simultaneously:

```bash
kubectl get events -n shopnow-ns --watch
```

Stop live logging with:

```text
Ctrl+C
```