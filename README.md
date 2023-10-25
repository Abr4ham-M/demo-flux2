# demo de flux2 

## Pre-requisitos

## Instalación Flux en MacOS

brew install fluxcd/tap/flux


## Instalación Flux en Kubernetes

### Bootstraping en Github

export GITHUB_TOKEN=<PAT-token>

flux bootstrap github \
  --token-auth \
  --owner=cvergarae \
  --repository=demo-flux2 \
  --branch=main \
  --path=clusters/demo-cluster \
  --personal

## Creación de un manifiesto.

flux create source git podinfo \
  --url=https://github.com/cvergarae/podinfo \
  --branch=master \
  --interval=1m \
  --export > ./clusters/demo-cluster/podinfo-source.yaml

## Creación de un manifiesto.

flux create kustomization podinfo \
  --target-namespace=default \
  --source=podinfo \
  --path="./kustomize" \
  --prune=true \
  --wait=true \
  --interval=30m \
  --retry-interval=2m \
  --health-check-timeout=3m \
  --export > ./clusters/demo-cluster/podinfo-kustomization.yaml

## ver estado de despliegue
flux get kustomizations --watch


docker build . -t podinfo  