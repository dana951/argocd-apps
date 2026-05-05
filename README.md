# argocd-apps

Argo CD bootstrap repository that manages application onboarding using the [App of Apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/) pattern.

This repository is part of the larger [`eks-gitops-platform`](https://github.com/dana951/eks-gitops-platform) portfolio.  
Its responsibility is to define which Argo CD applications exist and how they map to deployable workload definitions.

## What This Repo Does

- Defines a root Argo CD `Application` (`main-app`) that points to this repository.
- Uses `apps/` as the declarative catalog of managed applications.
- Uses an `ApplicationSet` to generate one Argo CD `Application` per environment for [`podinfo`](https://github.com/dana951/app-source.git) app.
- Enables automated sync, self-healing, and pruning for managed applications.

## Why This Repository Exists

This repository separates **Argo CD control-plane configuration** from **application manifests**:

- [`argocd-apps`](https://github.com/dana951/argocd-apps.git): which applications Argo CD should manage.
- [`gitops-manifests`](https://github.com/dana951/gitops-manifests.git): the Helm chart and environment values those applications deploy.

This split makes responsibilities clear: Argo CD application definitions stay here, and deployable workload configuration stays in `gitops-manifests`.

## Repository Layout

```text
argocd-apps/
├── main-app.yaml                   # Application - Root App-of-Apps entrypoint
└── apps/
    ├── podinfo-appset.yaml         # ApplicationSet
    └── podinfo-appset-pr-envs.yaml # ApplicationSet      

```

## App of Apps Bootstrap Flow

1. Apply `main-app.yaml` once to the Argo CD namespace.
2. Argo CD syncs this repository path (`apps/`).
3. `podinfo-appset.yaml` is applied for long-lived environments (staging/prod):
   - This `ApplicationSet` generates one Application per environment:
     - `podinfo-staging`
     - `podinfo-prod`
4. `podinfo-appset-pr-envs.yaml` is applied for ephemeral QA testing environments during CI/CD PR validation flow
   - This `ApplicationSet` watches [`gitops-manifests`](https://github.com/dana951/gitops-manifests.git) branches created during CI/CD PR validation flow and generates one Argo CD `Application` per matching branch.
5. Each generated app deploys `charts/podinfo` using the corresponding values file.

## Managed Application Behavior

Both the root app and generated apps are configured with:

- `automated.prune: true` - resources removed from Git are removed from cluster.
- `automated.selfHeal: true` - drift from desired state is automatically corrected.
- `CreateNamespace=true` (in generated apps) - target namespace is created if missing.

## Manual Quick Start Validation

Apply the root application (bootstrap step):

```bash
kubectl apply -f main-app.yaml -n argocd
```

Validate generated applications:

```bash
kubectl get applications -n argocd
```

You should see one `podinfo-*` Argo CD Application per environment directory found in [`gitops-manifests`](https://github.com/dana951/gitops-manifests.git).

## Onboarding a New Application

To onboard another service, add a new manifest under `apps/` (typically an `Application` or `ApplicationSet`) that points to the corresponding GitOps manifests repository/path.


## Related Repositories

- Platform overview: [`eks-gitops-platform`](https://github.com/dana951/eks-gitops-platform)
- Infrastructure (EKS, Jenkins, Argo CD): [`infra-aws`](https://github.com/dana951/infra-aws)
- GitOps workload definitions: [`gitops-manifests`](https://github.com/dana951/gitops-manifests)
- Application source and CI pipeline: [`app-source`](https://github.com/dana951/app-source)
- Test automation gates: [`tests-repo`](https://github.com/dana951/tests-repo)

## License

See `LICENSE` in this repository.
