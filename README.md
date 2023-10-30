# demo de flux2 

## Pre-requisitos

## Instalación Flux en MacOS

brew install fluxcd/tap/flux


## Instalación Flux en Kubernetes

### Bootstraping en Github

export GITHUB_TOKEN=<PAT-token>

flux bootstrap github \
  --components-extra=image-reflector-controller,image-automation-controller \
  --token-auth \
  --owner=cvergarae \
  --repository=demo-flux2 \
  --branch=main \
  --path=clusters/demo-cluster \
  --read-write-key \
  --personal

## Creación de un manifiesto.

flux bootstrap github \
  --components-extra=image-reflector-controller,image-automation-controller \
  --owner=$GITHUB_USER \
  --repository=flux-image-updates \
  --branch=main \
  --path=clusters/my-cluster \
  --read-write-key \
  --personal

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


➜  polimatas flux reconcile kustomization podinfo -n flux-system




# Creación Política de inagen

flux create image policy podinfo \
--image-ref=podinfo \
--filter-regex='dev-*' \
--select-alpha='asc' \
--export > ./clusters/demo-cluster/podinfo-policy.yaml



flux create image repository podinfo \
--image=layer0/podinfo \
--interval=5m \
--export > ./clusters/demo-cluster/podinfo-registry.yaml

# Creación ImageUpdateAutomation



flux create image update flux-system \
--interval=30m \
--git-repo-ref=flux-system \
--git-repo-path="./clusters/demo-cluster" \
--checkout-branch=main \
--push-branch=main \
--author-name=fluxcdbot \
--author-email=fluxcdbot@users.noreply.github.com \
--commit-template="{{range .Updated.Images}}{{println .}}{{end}}" \
--export > ./clusters/demo-cluster/flux-system-automation.yaml

flux create image update flux-system \
--interval=30m \
--git-repo-ref=podinfo \
--git-repo-path="./kustomize" \
--checkout-branch=master \
--push-branch=master \
--author-name=fluxcdbot \
--author-email=fluxcdbot@users.noreply.github.com \
--commit-template="{{range .Updated.Images}}{{println .}}{{end}}" \
--export > ./clusters/demo-cluster/podinfo-automation.yaml


flux reconcile kustomization --with-source flux-system


---
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: podinfo
  namespace: flux-system
spec:
  filterTags:
    pattern: '^dev-(?P<ts>.*)'
    extract: '$ts'
  policy:
    numerical:
      order: asc
  imageRepositoryRef:
    name: podinfo