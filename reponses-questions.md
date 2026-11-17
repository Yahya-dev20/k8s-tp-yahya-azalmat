# Réponses aux questions — TP 4 Kubernetes

## 1. Rôle des Pods de kube-system

| Composant | Rôle |
| --- | --- |
| etcd | Stocke l'état et la configuration du cluster. |
| kube-apiserver | Expose l'API Kubernetes et traite les requêtes. |
| kube-scheduler | Affecte les nouveaux Pods aux nœuds. |
| kube-controller-manager | Exécute les contrôleurs qui rapprochent l'état réel de l'état souhaité. |
| kube-proxy | Met en œuvre le routage réseau des Services. |
| CoreDNS | Résout les noms DNS internes. |
| kindnet | Assure la connectivité réseau des Pods dans kind. |

## 2. Pourquoi le Pod web n'est-il pas recréé ?

Le Pod web est autonome : aucun contrôleur ne maintient son existence. Les Pods whoami sont gérés par un ReplicaSet, piloté par le Deployment, qui maintient le nombre de réplicas demandé. Un Pod whoami supprimé est donc remplacé.

## 3. Relation entre Deployment, ReplicaSet et Pod

Le Deployment gère les versions du modèle de Pod. Chaque ReplicaSet maintient les Pods correspondant à un modèle.

Trois ReplicaSets sont observés à la fin :

- v1.10 : zéro réplique ;
- v9.99 : zéro réplique ;
- v1.11 : six répliques prêtes.

Le rollback réutilise le ReplicaSet v1.11 et lui attribue la révision la plus récente. Les anciens modèles sont conservés pour permettre des retours de version. L'historique final présente les révisions 1, 3 et 4.

## 4. Comment le Service trouve-t-il les Pods ?

Le sélecteur du Service correspond au label `app=whoami` des Pods. Kubernetes maintient les EndpointSlices avec les adresses et les conditions de disponibilité des endpoints. Le trafic est dirigé vers les endpoints utilisables. Le relevé final présente six adresses associées au Service.

## 5. Disponibilité pendant l'image cassée et maxUnavailable

Pour six réplicas, les valeurs par défaut donnent :

- `maxUnavailable: 25%` : un Pod après arrondi inférieur ;
- `maxSurge: 25%` : deux Pods supplémentaires après arrondi supérieur.

Pendant l'essai de l'image inexistante v9.99, les nouveaux Pods échouent avec ErrImagePull / ImagePullBackOff. Cinq anciens Pods v1.11 restent prêts et continuent à répondre. Le remplacement ne peut pas se terminer tant que les nouveaux Pods ne deviennent pas disponibles. Le rollback rétablit ensuite six réplicas v1.11.

Une erreur HTTP `Connection reset by peer` a toutefois été mesurée pendant la mise à jour valide v1.10 vers v1.11, avant l'essai de l'image cassée. On ne peut donc pas conclure à une absence totale d'interruption sur toute la session. Le suivi par sondes toutes les 0,5 seconde ne mesure pas les intervalles entre requêtes.

## Défi DNS

Le test BusyBox vers `http://whoami.default.svc.cluster.local` a retourné une réponse HTTP de whoami.

- `whoami` : nom du Service ;
- `default` : namespace ;
- `svc` : catégorie des Services ;
- `cluster.local` : domaine DNS du cluster.

Le Pod temporaire a été supprimé après le test.
