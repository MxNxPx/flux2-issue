# flux2-issue

# flux2-issue

Steps to reproduce flux issue
```sh
export GITHUB_TOKEN GITHUB_REPO GITHUB_USER GITHUB_PWD

time (kind create cluster --image kindest/node:v1.19.7 --name staging --wait 5m && kubectl wait --timeout=5m --for=condition=Ready nodes --all)

kubectl config use-context kind-staging
kubectl config current-context
flux check --pre

flux bootstrap git \
    --token-auth=true \
    --context=kind-staging \
    --url=https://github.com/${GITHUB_USER}/${GITHUB_REPO} \
    --username=${GITHUB_USER} \
    --password=${GITHUB_PWD} \
    --branch=main \
    --path=clusters/staging \
    --verbose

flux get sources all; kubectl get ks,hr; kubectl get po -A
```


Straight kustomize with 'kubectl apply -f' also fails

```sh
## cd to the problem ks
cd clusters/staging/cm-test

## cleanup in case it exists
kubectl get cm -n kube-system -l grafana_dashboard=1
kubectl delete -n kube-system cm -l grafana_dashboard=1

## run it with apply
kustomize build . | kubectl apply --force=true -f -
kubectl get cm -n kube-system -l grafana_dashboard=1
#configmap/grafana-dashboard-1 created
#configmap/grafana-dashboard-2 created
#The ConfigMap "grafana-dashboard-3" is invalid: metadata.annotations: Too long: must have at most 262144 bytes
```


But kustomize with 'kubectl replace --force=true -f' WORKS

```sh
## cleanup in case it exists
kubectl get cm -n kube-system -l grafana_dashboard=1
kubectl delete -n kube-system cm -l grafana_dashboard=1

## run it with replace
kustomize build . | kubectl replace --force=true -f -
kubectl get cm -n kube-system -l grafana_dashboard=1
#configmap/grafana-dashboard-1 replaced
#configmap/grafana-dashboard-2 replaced
#configmap/grafana-dashboard-3 replaced

## run it again with replace
kustomize build . | kubectl replace --force=true -f -
kubectl get cm -n kube-system -l grafana_dashboard=1
#configmap "grafana-dashboard-1" deleted
#configmap "grafana-dashboard-2" deleted
#configmap "grafana-dashboard-3" deleted
#configmap/grafana-dashboard-1 replaced
#configmap/grafana-dashboard-2 replaced
#configmap/grafana-dashboard-3 replaced
```
