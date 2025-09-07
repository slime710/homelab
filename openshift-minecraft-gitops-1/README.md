# OpenShift GitOps + Tekton: Auto-Deploy Minecraft

## 1) Install Argo CD operator & instance
oc apply -f operators/openshift-gitops/subscription.yaml
oc apply -f bootstrap/argocd/namespace.yaml
oc apply -f bootstrap/argocd/instance.yaml

## 2) Bootstrap GitOps (App-of-Apps)
# In bootstrap/argocd/app-of-apps.yaml die repoURL auf dein Repo ändern
oc apply -f bootstrap/argocd/app-of-apps.yaml

## 3) Minecraft App
# Argo CD synced clusters/dev/argocd-apps.yaml -> apps/minecraft/overlays/dev
# Danach Service-External IP/Hostname auf Port 25565 prüfen.

## 4) Tekton CI
oc apply -f pipelines/tekton/namespace.yaml
oc apply -f pipelines/tekton/sa-rbac.yaml
oc apply -f pipelines/tekton/pipeline.yaml
oc apply -f pipelines/tekton/triggers.yaml

# Webhook-URL holen:
oc get route -n ci git-listener -o jsonpath='{.spec.host}'

## Hinweise
- Service ist type LoadBalancer (TCP 25565). Bei Bedarf auf NodePort wechseln.
- PVC 10Gi speichert World-Daten. StorageClass ggf. setzen.
- Läuft als non-root, fsGroup für Schreibrechte.
- Für Prod ein zweites Overlay mit höheren Limits & Backups anlegen.