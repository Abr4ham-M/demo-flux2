# demo de flux2 

## Pre-requisitos

## Instalación Flux en MacOS

brew install fluxcd/tap/flux


## Instalación Flux en Kubernetes

### Bootstraping en Github

export GITHUB_TOKEN=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

flux bootstrap github \                                     
  --token-auth \
  --owner=cvergarae \
  --repository=demo-flux2 \
  --branch=main \
  --path=clusters/demo-cluster \
  --personal

## Creación de un manifiesto.

flux create source git podinfo \
  --url=https://github.com/stefanprodan/podinfo \
  --branch=master \
  --interval=1m \
  --export > ./clusters/demo-cluster/podinfo-source.yaml

  

