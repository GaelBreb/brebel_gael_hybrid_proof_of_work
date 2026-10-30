Ticket — accès `kubectl` depuis un poste local via l'ingress controller (SNI passthrough).

- Vérification de la faisabilité (Service `kubernetes`, accès réseau depuis les pods de l'ingress controller)
- Écriture du routeur `IngressRouteTCP` et de l'Application ArgoCD associée
- Validation avant push : CRD, `kubectl apply --dry-run=server`, résolution DNS
- Rédaction du runbook en français et lien depuis le README
- [Commit `e493506` — 5 fichiers, 228 lignes ; PR non mergée](https://github.com/EpitechMscProPromo2027/T-CLO-901-TLS-9-1-kube-4/pull/28/changes/e49350639b159edf700fae8c227a18af254604e4)

Review de PR pour kubequest
Relance pour les PR en cours pour l'ESP

Branche `docs/translate-provision-docs-to-french`.

- Traduction en français des deux derniers documents anglais du dépôt
- Audit des liens Markdown : 2 liens morts corrigés
- Correction de 3 défauts supplémentaires, dont un bloquant (la procédure d'init du control plane ne pouvait pas aboutir en l'état)
- [commit `a8d1d72c8cf90711f244931da1ff6201a1bde135`](https://github.com/EpitechMscProPromo2027/T-CLO-901-TLS-9-1-kube-4/pull/29/changes/a8d1d72c8cf90711f244931da1ff6201a1bde135)
- Lecture de documentation pour l'implémentation de Keycloak dans notre infra Kubequest

## Reste à faire

- Merger CLO-24, puis valider `kubectl get nodes` depuis un poste local
- Restreindre l'accès à l'API server par liste blanche d'IP
- Ajuster les timeouts TCP des deux étages Traefik
- Mettre en place Keycloak comme fournisseur OIDC
- Externaliser l'Elastic IP, codée en dur dans plusieurs manifestes
- Créer les workflows GitHub Actions
