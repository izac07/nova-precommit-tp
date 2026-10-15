# Indicateurs de suivi — NOVA SERVICES

Ces trois indicateurs servent à vérifier que les contrôles mis en place sont vraiment utilisés et qu'ils servent à quelque chose.

## Indicateur 1 — Taux de dépôts configurés

**Ce qu'il mesure** : la part des dépôts Python qui ont un `.pre-commit-config.yaml` versionné avec les contrôles obligatoires (Bandit, detect-secrets, pytest, pip-audit).

**Calcul** : nombre de dépôts configurés / nombre total de dépôts Python × 100

| Élément | Valeur |
|---|---|
| Source | Liste des dépôts de l'organisation GitHub, avec une vérification de la présence du fichier de configuration |
| Propriétaire | Responsable sécurité |
| Fréquence | Une fois par mois |
| Seuil d'alerte | Moins de 90 % |
| Action | Relancer les équipes concernées et les aider à installer les hooks sur les dépôts manquants |

**Pourquoi** : si un dépôt n'a pas les hooks, rien n'empêche un secret ou un `eval()` d'y entrer, comme on l'a vu au début du TP.

## Indicateur 2 — Délai médian de correction

**Ce qu'il mesure** : le temps entre la détection d'un problème (vulnérabilité de dépendance, problème Bandit) et sa correction vérifiée.

**Calcul** : médiane des délais entre la date de détection et la date où le contrôle repasse au vert

| Élément | Valeur |
|---|---|
| Source | Sorties de pip-audit et Bandit, historique des commits de correction |
| Propriétaire | Tech lead de chaque dépôt |
| Fréquence | Une fois par mois |
| Seuil d'alerte | Plus de 7 jours si un correctif existe, plus de 30 jours pour une faille grave sans correctif |
| Action | Mettre la correction en priorité dans le sprint et prévenir le responsable sécurité |

**Pourquoi** : détecter une faille ne sert à rien si elle reste ouverte longtemps. Dans le TP, la faille PYSEC-2026-2270 de python-dotenv a été corrigée tout de suite en passant à la version 1.2.4, parce qu'un correctif existait déjà.

## Indicateur 3 — Nombre d'exceptions actives

**Ce qu'il mesure** : le nombre de contrôles volontairement désactivés à un endroit : entrées dans `.secrets.baseline`, `# pragma: allowlist secret`, `# nosec` pour Bandit, `--ignore-vuln` pour pip-audit.

**Calcul** : nombre total d'exceptions présentes dans les dépôts, avec le nombre d'exceptions expirées à part

| Élément | Valeur |
|---|---|
| Source | Recherche de ces marqueurs dans les dépôts et registre des exceptions validées |
| Propriétaire | Responsable sécurité |
| Fréquence | Une fois par mois |
| Seuil d'alerte | Une exception expirée, ou une hausse de plus de 20 % d'un mois sur l'autre |
| Action | Revoir chaque exception : la supprimer, la corriger ou la renouveler avec une nouvelle justification |

**Pourquoi** : une exception mal utilisée peut cacher un vrai secret ou une vraie faille. Moins il y en a, mieux c'est. Sur notre dépôt, la baseline est vide.

## Lecture des indicateurs

Un nombre élevé de détections ne veut pas forcément dire que la sécurité se dégrade. Ça peut aussi vouloir dire que les contrôles sont mieux déployés et trouvent plus de choses. Il faut toujours lire ces chiffres avec leur contexte.
