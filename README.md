# flux2-issue

Steps to reproduce issue
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
