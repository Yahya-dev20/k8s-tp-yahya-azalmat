# TP 4 Kubernetes — ENSAH

**Yahya Azalmat — Génie Informatique / Génie logiciel — 2026/2027**

Ce dépôt contient les fichiers nécessaires à la remise du TP : cluster kind à trois nœuds, Pod Nginx, Deployment whoami, Service NodePort, scaling, rolling update, rollback et test DNS.

## Fichiers

| Fichier | Contenu |
| --- | --- |
| `main.tex` | Rapport LaTeX complet, explications de 49 commandes, consoles intégrées et réponses aux cinq questions |
| `logos/` | Logos ENSAH et Université Abdelmalek Essaâdi de la page de garde |
| `kind-config.yaml` | Un control-plane, deux workers, port hôte 30080 |
| `pod.yaml` | Pod autonome Nginx |
| `whoami.yaml` | Deployment initial de trois réplicas v1.10 et Service NodePort 30080 |
| `historique-rollout.txt` | Historique réellement obtenu après rollback |

## Compiler le rapport dans Overleaf

Importer tout le ZIP du dossier, conserver le sous-dossier `logos`, puis choisir `main.tex` comme document principal et **pdfLaTeX** comme compilateur. Compiler deux fois pour actualiser la table des matières. Les consoles sont dessinées par LaTeX : aucun dossier `captures` n'est nécessaire. Les logos sont optionnels pour la compilation, mais doivent être importés pour apparaître sur la couverture.

## Reproduire les premières étapes

Avec Docker, kind et kubectl installés, depuis le dossier du projet dans un terminal Bash / Ubuntu WSL2 :

```bash
kind create cluster --config kind-config.yaml
kubectl get nodes -o wide
kubectl apply -f pod.yaml
kubectl apply -f whoami.yaml
kubectl rollout status deployment/whoami
curl http://localhost:30080
```

Les commandes et les résultats de l'ensemble du TP sont présentés dans `main.tex`. Les YAML décrivent l'état initial : trois réplicas et whoami v1.10. Après les manipulations, l'état observé est de six réplicas v1.11. Réappliquer le YAML initial peut donc modifier cet état.

## Observations de l'exécution

- Trois nœuds Ready ; Pod Nginx fonctionnel ; auto-réparation et scaling validés.
- Image v9.99 introuvable : ErrImagePull / ImagePullBackOff ; les anciens Pods restent disponibles pendant cet essai.
- Rollback réussi vers v1.11, six réplicas ; trois ReplicaSets conservés.
- Test DNS BusyBox réussi vers `whoami.default.svc.cluster.local`.
- Une réinitialisation de connexion HTTP a été observée pendant la mise à jour valide v1.10 vers v1.11 ; le rapport la mentionne.

Les consoles sont des rendus reconstitués des sorties réellement enregistrées dans Ubuntu WSL2, avec une présentation inspirée de PowerShell. Ce ne sont pas des captures natives. Les symboles décoratifs ont été normalisés pour pdfLaTeX.

Le kubeconfig, les identifiants d'accès, les binaires et les fichiers temporaires ne font pas partie de la remise.
