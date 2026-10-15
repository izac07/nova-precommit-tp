# Politique de sécurisation du code — NOVA SERVICES

## 1. Pourquoi cette politique

En faisant le TP, on a vu qu'un mot de passe en dur et un `eval()` sur une saisie utilisateur pouvaient passer très facilement dans un dépôt. Le test avec `__import__('os').getcwd()` a suffi à exécuter du code à la place d'un calcul. Cette politique fixe les règles pour que ce genre d'erreur soit arrêté avant d'arriver dans Git.

Elle s'applique à tous les dépôts Python de NOVA SERVICES : l'API, l'outil d'administration et les scripts d'automatisation.

## 2. Environnement de référence

- Python 3.13 minimum. Python 3.7 est interdit : il n'est plus maintenu et les versions récentes des hooks ne s'installent pas dessus.
- Un environnement virtuel `.venv` par projet, jamais versionné.
- Les outils sont listés dans `requirements-dev.txt` pour que tout le monde ait le même environnement.

## 3. Contrôles obligatoires

| Contrôle | Outil | Ce qu'il vérifie |
|---|---|---|
| Hygiène des fichiers | pre-commit-hooks | Espaces en fin de ligne, fin de fichier, syntaxe YAML, fichiers trop gros |
| Analyse du code | Bandit (`-ll`) | Pratiques dangereuses comme `eval()` |
| Secrets | detect-secrets + `.secrets.baseline` | Mots de passe, clés et tokens dans les fichiers |
| Tests | pytest | Le code fonctionne toujours après une modification |
| Dépendances | pip-audit | Les bibliothèques n'ont pas de vulnérabilité connue |

Bandit avec `-ll` ne bloque que les problèmes de sévérité Medium ou plus. Dans le TP, il a vu le mot de passe en dur mais l'a classé Low et ne l'a pas bloqué. C'est detect-secrets qui l'a arrêté. Les deux outils sont donc obligatoires ensemble, l'un ne remplace pas l'autre.

## 4. Contrôles en local et en CI

**En local (pre-commit, à chaque commit)** : tous les contrôles du tableau. Chaque développeur doit lancer `pre-commit install` après avoir cloné le dépôt.

**En CI** : un hook local peut être contourné, par exemple avec `git -c core.hooksPath=/dev/null commit`. Le commit passe alors sans aucun contrôle. La CI doit donc rejouer `pre-commit run --all-files` à chaque push et à chaque pull request. Elle n'est pas encore en place sur ce dépôt et c'est la prochaine étape.

**Côté serveur (GitHub)** : la branche `main` doit être protégée. Pas de push direct, merge uniquement par pull request, CI verte obligatoire et au moins une relecture.

## 5. Quand un commit est bloqué

Un commit est refusé si :
- detect-secrets trouve un secret qui n'est pas dans la baseline ;
- Bandit trouve un problème Medium ou High ;
- un test échoue ;
- pip-audit trouve une vulnérabilité connue sans exception validée ;
- un hook a corrigé un fichier et la correction n'a pas été rajoutée avec `git add`.

Il n'est pas autorisé de contourner un hook (`--no-verify`, `core.hooksPath`) pour forcer un commit.

## 6. Secrets et faux positifs

- Aucun secret dans le code. Les valeurs vont dans `.env`, qui est dans le `.gitignore`. Le dépôt contient seulement `.env.example`, sans valeur.
- Avant de déclarer un faux positif, on vérifie que ce n'est pas un vrai secret.
- Si c'est un vrai secret : on le retire du code, on le révoque ou on le change tout de suite, et s'il a été poussé, on nettoie l'historique. Le mettre dans la baseline ne le protège pas, il reste lisible dans le code et dans Git.
- Si c'est un faux positif : on le marque sur la ligne avec `# pragma: allowlist secret` et on explique pourquoi dans le commit.
- La baseline doit rester la plus petite possible. Sur ce dépôt, elle est vide.

## 7. Dépendances vulnérables

Quand pip-audit trouve une faille, on regarde d'abord si elle touche vraiment notre code. Par exemple, python-dotenv 1.0.1 avait la faille PYSEC-2026-2270 sur `set_key()` et `unset_key()`. On n'utilise que `load_dotenv()`, donc on n'était pas exposés, mais on a quand même mis à jour vers la 1.2.4 parce que le correctif existait.

S'il n'y a pas de correctif :
- on vérifie si la partie vulnérable est utilisée ;
- on cherche une alternative ou une mesure qui limite le risque ;
- on peut ignorer temporairement la faille avec `pip-audit --ignore-vuln <ID>`, mais seulement avec une exception validée et une date de fin.

## 8. Exceptions

Toute exception (faux positif, `# nosec`, faille ignorée) doit indiquer :
- le contrôle concerné ;
- la raison ;
- le risque qui reste ;
- une date de fin, au maximum 90 jours.

Elle est validée par le responsable sécurité. Pour une dépendance vulnérable sans correctif, il faut aussi l'accord du tech lead. Les exceptions sont revues chaque mois, et une exception expirée redevient bloquante.

## 9. Mise à jour des outils

- Les versions des hooks sont fixées dans `.pre-commit-config.yaml` pour que tout le monde ait les mêmes résultats.
- Le tech lead lance `pre-commit autoupdate` une fois par mois, dans un commit séparé, puis vérifie que tous les contrôles passent.
- Si un outil reçoit un correctif de sécurité, on le met à jour sous 7 jours.

## 10. Limites

Les hooks ne remplacent pas la relecture de code, les tests de sécurité plus poussés, ni un vrai gestionnaire de secrets en production. Un contrôle qui passe ne prouve pas qu'il n'y a aucune faille, il prouve seulement que les problèmes recherchés n'ont pas été trouvés.
