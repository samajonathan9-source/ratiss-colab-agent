# RATISS Colab Agent Control Plane

Ce dépôt contient un notebook Google Colab pour piloter un agent de développement avec une intégration **GitHub globale**. Le notebook ne dépend d’aucun dépôt précis : tu peux sélectionner un dépôt existant accessible par ton compte GitHub ou en créer un nouveau.

## Liens rapides

- **Ouvrir le notebook dans Colab :** [RATISS_Colab_Agent_Control_Plane.ipynb](https://colab.research.google.com/github/samajonathan9-source/ratiss-colab-agent/blob/main/RATISS_Colab_Agent_Control_Plane.ipynb)
- **Dépôt GitHub :** [samajonathan9-source/ratiss-colab-agent](https://github.com/samajonathan9-source/ratiss-colab-agent)

## Important avant de commencer

Colab fournit un environnement temporaire. Le notebook peut être interrompu lorsque le runtime expire ou lorsque le GPU n’est plus disponible. Le notebook sauvegarde donc l’état de la tâche dans un fichier de checkpoint et reprend la préparation lorsqu’il est relancé.

Colab ne garantit pas le réveil automatique d’un runtime complètement arrêté. Pour une reprise après expiration, il faut rouvrir le notebook, reconnecter le runtime et relancer les cellules. Les modifications déjà poussées sur GitHub restent disponibles.

## 1. Créer le token GitHub

Le notebook demande un **Personal Access Token GitHub**, appelé PAT.

| Format | Type | Utilisation |
|---|---|---|
| `ghp_...` | PAT classique | Le plus simple pour une intégration générale GitHub |
| `github_pat_...` | Fine-grained PAT | Plus précis et recommandé pour limiter les permissions |

Le token doit appartenir au compte GitHub qui possède ou peut modifier les dépôts RATISS.

### PAT classique

Dans GitHub : **Settings → Developer settings → Personal access tokens → Tokens (classic)**. Active au minimum la permission `repo`. Elle permet notamment de cloner, modifier et pousser vers des dépôts privés. Pour créer de nouveaux dépôts, utilise un compte autorisé à effectuer cette opération.

### Fine-grained PAT

Dans GitHub : **Settings → Developer settings → Personal access tokens → Fine-grained tokens**. Choisis le compte propriétaire, puis les dépôts accessibles. Active au minimum :

- **Contents: Read and write** ;
- **Metadata: Read** ;
- **Pull requests: Read and write** si le notebook doit créer des Pull Requests ;
- la permission de création de dépôt uniquement si nécessaire.

Donne au token une expiration raisonnable. Ne publie jamais le token dans le notebook, dans un commit, dans une capture d’écran ou dans un message.

## 2. Ouvrir et lancer le notebook

1. Ouvre le [lien Colab direct](https://colab.research.google.com/github/samajonathan9-source/ratiss-colab-agent/blob/main/RATISS_Colab_Agent_Control_Plane.ipynb).
2. Connecte-toi à Google si Colab le demande.
3. Accepte la copie du notebook dans ton espace Colab si nécessaire.
4. Lance les cellules dans l’ordre, de haut en bas.
5. Autorise l’installation des dépendances et l’accès au runtime.

La première cellule installe `git`, `gh`, `PyGithub`, `ipywidgets` et `requests`.

## 3. Saisir le token GitHub

Sur ordinateur, tu peux utiliser la zone **Secrets** de Colab avec un secret nommé exactement `GITHUB_TOKEN`.

Sur mobile, la zone Secrets peut ne pas apparaître. Exécute la cellule de configuration et colle le PAT lorsqu’elle affiche :

```text
GitHub Fine-grained token (hidden):
```

Le champ masque la saisie. Appuie sur **Entrée** ou sur la touche de validation du clavier. Le notebook doit ensuite afficher :

```text
Connecté à GitHub comme: ton_nom_github
```

Si la cellule attend indéfiniment, arrête-la et relance uniquement la cellule de configuration. Vérifie que tu as bien collé le token dans le champ masqué.

## 4. Vérifier la connexion

La cellule de diagnostic vérifie l’identité GitHub avec l’API. Elle ne montre jamais le token.

```text
GitHub OK: samajonathan9-source
Le dépôt privé sera cloné avec le token via un header temporaire.
```

| Message | Correction |
|---|---|
| `GITHUB_TOKEN est vide` | Recréer le secret ou saisir le PAT dans le champ masqué |
| `Token GitHub refusé` | Vérifier l’expiration et les permissions du PAT |
| `Resource not accessible` | Ajouter Contents et Metadata |
| `Repository not found` | Vérifier `propriétaire/dépôt` et l’accès du compte |
| `could not read Username` | Relancer le notebook mis à jour et vérifier le PAT |

## 5. Choisir un dépôt existant

Dans l’interface GitHub, écris le dépôt sous la forme :

```text
proprietaire/nom-du-depot
```

Exemples :

```text
samajonathan9-source/ratiss-bio
samajonathan9-source/Crypto-net-veo-
ratiss-labs/ratiss-qpu-ambient
```

Écris ensuite une branche de travail, par exemple :

```text
agent/diagnostic-auth
```

Clique sur **Sélectionner**, puis sur **Cloner / préparer branche**. Le notebook clone le dépôt dans `/content/ratiss-agent/workspace/` et prépare une branche dédiée. Il ne pousse pas directement sur `main` par défaut.

## 6. Créer un nouveau dépôt

Dans la même interface : écris le nom, indique éventuellement une organisation, choisis privé ou public, puis clique sur **Créer dépôt**. Sélectionne ensuite le dépôt créé et prépare sa branche.

La création nécessite une permission GitHub adaptée. Si elle est refusée, crée le dépôt manuellement sur GitHub puis utilise sa référence `proprietaire/nom-du-depot` dans le notebook.

## 7. Travailler dans le dépôt

Le notebook permet de consulter le statut Git, vérifier la branche active, lancer des tests, exécuter des scripts autorisés, inspecter les fichiers et sauvegarder un checkpoint. Les commandes sont filtrées par une liste blanche.

```python
status()
checkpoint('tested', 'Tests locaux terminés')
```

## 8. Commit, push et Pull Request

Avant de pousser, vérifie le diff, exécute les tests, sauvegarde un checkpoint et confirme la branche active.

```python
commit_and_push('feat: complete RATISS validation')
```

Pour ouvrir une Pull Request :

```python
open_pull_request(
    'Complete RATISS validation',
    'Tests exécutés dans Colab. Merci de revoir le diff avant fusion.'
)
```

La Pull Request doit être relue avant fusion. Le notebook ne fusionne pas automatiquement la Pull Request.

## 9. Reprendre après expiration de Colab

Lorsque Colab s’arrête :

1. rouvre le notebook ;
2. reconnecte le runtime ;
3. relance les cellules d’installation et de configuration ;
4. saisis à nouveau le PAT si aucun secret Colab n’est disponible ;
5. exécute la cellule **Checkpoint et reprise** ;
6. vérifie le dépôt et la branche ;
7. reclone le dépôt si `/content` a été effacé ;
8. reprends la phase indiquée par le checkpoint.

La reprise s’appuie sur :

```text
/content/ratiss-agent/state.json
```

Ce fichier est temporaire dans Colab. Pour conserver l’état malgré une nouvelle session, pousse régulièrement les changements sur une branche GitHub ou copie les résultats dans un stockage persistant.

## 10. Cycle complet recommandé

```text
Ouvrir Colab
  → Installer les dépendances
  → Fournir GITHUB_TOKEN
  → Vérifier l’identité GitHub
  → Lister ou sélectionner un dépôt
  → Créer un dépôt si nécessaire
  → Créer une branche de travail
  → Cloner
  → Exécuter la tâche
  → Sauvegarder un checkpoint
  → Tester
  → Relire le diff
  → Commit et push
  → Créer une Pull Request
  → Revue humaine
  → Fusion sur GitHub
```

## 11. Sécurité

Ne colle jamais un PAT dans une cellule Markdown. Ne l’inclus pas dans une URL Git. Ne l’imprime pas avec `print`. Ne le committe jamais. Si le token apparaît dans un log ou une capture, révoque-le immédiatement dans GitHub et crée-en un nouveau.

Le notebook utilise une authentification Git temporaire par header HTTP pour éviter de placer le token dans l’URL de clonage. Le runtime Colab reste néanmoins temporaire.

## 12. Limite du modèle OpenHands-LM-32B

Le notebook prépare le runtime, GitHub et le contrôle des tâches. Le chargement local d’un modèle 32B dépend de la mémoire GPU disponible. Une T4 ou une P100 ne suffit généralement pas pour une version complète en précision standard. Il faut utiliser une quantification compatible, un runtime distribué ou un endpoint distant.

La cellule GPU affiche le matériel disponible avant tout lancement lourd.

## 13. Dépannage rapide

| Problème | Action immédiate |
|---|---|
| La cellule attend le token | Coller le PAT dans le champ masqué puis appuyer sur Entrée |
| Aucun espace Secrets sur mobile | Utiliser la saisie masquée directe |
| `could not read Username` | Utiliser la version publique mise à jour et vérifier le PAT |
| Dépôt privé inaccessible | Ajouter Contents: Read and write |
| Push refusé | Vérifier Contents: Read and write et la branche ciblée |
| Pull Request refusée | Ajouter Pull requests: Read and write |
| Création de dépôt refusée | Ajouter la permission de création ou créer manuellement |
| Runtime expiré | Relancer les cellules, recharger le checkpoint et recloner |
| GPU absent | Reconnecter le runtime ou choisir CPU/endpoint distant |

## Licence et responsabilité

Le notebook est fourni comme outil de développement et de prototypage. Vérifie chaque modification avant de la pousser ou de la fusionner. Les permissions GitHub doivent rester limitées au périmètre réellement nécessaire.
