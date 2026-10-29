# MediTriage · repositorio GitOps

Estado deseado del clúster para MediTriage. Argo CD sincroniza este repo con Kubernetes (modelo *pull*).

## Estructura

```
base/              manifiestos comunes: deployment, service, ingress, configmap, hpa
overlays/dev/      namespace meditriage; imagen por digest (la actualiza el CI vía PR)
overlays/prod/     namespace meditriage-prod; HPA 3 a 10 réplicas (solo estructura)
```

Previsualizar lo que se aplicaría:

```
kubectl kustomize overlays/dev
kubectl kustomize overlays/prod
```

## Reglas

- La imagen se referencia **siempre por digest** (`@sha256:...`), nunca `:latest`.
- No hay push directo a `main`: todo cambio entra por Pull Request.
- **dev:** el pipeline del repo de la app abre un PR con el digest nuevo (`kustomize edit set image`).
- **prod:** se promueve con un PR manual, con 2 aprobaciones.
- El Deployment no declara `replicas`: el HPA controla el número de pods (si estuviera fijo, Argo con `selfHeal` revertiría el escalado).
- El nombre de imagen en `base/` (`ghcr.io/guffy0921/meditriage`) no se cambia: el CI lo sustituye por el de la organización al promover.

## Origen

Los manifiestos provienen de `k8s/` del repo de la aplicación (S07).
