# DevOps TP11 - GitLab CI/CD et runner auto-hébergé
# DevOps TP11 - GitLab CI/CD et Runner auto-hébergé

🔗 **Lien vers le projet GitLab :** [https://gitlab.com/aarijbelaid/contact-api-tp11](https://gitlab.com/aarijbelaid/contact-api-tp11)

⚠️ **Note :** L'exécution des pipelines sur GitLab.com a été bloquée par la politique de vérification d'identité (anti-spam) de GitLab pour les nouveaux comptes. Cependant, le fichier `.gitlab-ci.yml` a été complété et validé syntaxiquement, et le runner auto-hébergé a été installé et enregistré avec succès.

## 📸 Captures d'écran

### 1. Runner en ligne
Le runner `lab-runner` est enregistré et en ligne (vert) dans GitLab.
![Runner en ligne](captures/runner-online.png)

### 2. Validation du pipeline
L'éditeur de pipeline GitLab confirme que la syntaxe est correcte.
![Pipeline Editor](captures/pipeline-editor.png)

### 3. Variable masquée et protégée
La variable `DEMO_TOKEN` est configurée avec les options `Masked` et `Protected`.
![Variable DEMO_TOKEN](captures/variable-demo-token.png)

## ⚙️ Réponses aux questions du TP

### Pourquoi aucun port n'a dû être ouvert sur la VM ?
Le runner fonctionne selon un **modèle pull** : c'est lui qui interroge GitLab (en HTTPS, port 443) pour récupérer les jobs. GitLab ne se connecte jamais directement au runner. Il n'y a donc pas besoin d'ouvrir de port entrant sur le pare-feu de la VM.

### Quel risque prend-on en montant le socket Docker ?
Monter `/var/run/docker.sock` dans le conteneur du runner donne à ce dernier un accès root complet au démon Docker de la machine hôte. Si un job (ou une dépendance compromise) est malveillant, il peut lancer un conteneur privilégié et s'échapper du conteneur pour prendre le contrôle total de la VM.

### Cache et Artifacts
- **Cache** : Le job `install` utilise un cache basé sur `package-lock.json` pour réutiliser le dossier `.npm/` entre les pipelines.
- **Artifacts** : Le job `install` transmet `node_modules/` (1h), et `unit-tests` transmet le rapport JUnit (`reports/junit.xml`).

### Rules
- `build-image` : S'exécute sur les Merge Requests et la branche par défaut.
- `push-image` : S'exécute uniquement sur la branche par défaut (`$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH`).

### Variable DEMO_TOKEN
Grâce à l'option `Masked`, la valeur est remplacée par `[MASKED]` dans les logs. Grâce à `Protected`, elle n'est injectée que dans les pipelines des branches protégées (comme `main`), ce qui évite qu'une branche `feature/...` puisse voler le secret.

## 📝 Tableau comparatif

| Critère | Jenkins (TP9) | GitHub Actions (TP10) | GitLab CI/CD (TP11) |
| :--- | :--- | :--- | :--- |
| **Configuration** | Fichier `Jenkinsfile` (Groovy) | YAML (`.github/workflows`) | YAML (`.gitlab-ci.yml`) |
| **Runners / agents** | Agents auto-hébergés | Hébergés par GitHub ou auto-hébergés | Hébergés par GitLab ou auto-hébergés |
| **Secrets** | Credentials Store | Secrets GitHub | Variables CI/CD (masquées/protégées) |
| **Sécurité intégrée** | Plugins (nécessite config) | CodeQL, Dependabot (intégré) | SAST, Secret Detection (templates inclus) |
| **Maintenance** | Élevée | Faible (SaaS) | Moyenne (SaaS) / Élevée (auto-hébergé) |
| **Intégration Git** | Faible (via webhooks) | Excellente | Excellente |

## 🎯 Recommandation pour une PME

Pour une PME qui démarre, **GitHub Actions ou GitLab CI/CD** sont recommandés. Ils sont "SaaS" (pas de serveur à maintenir) et intègrent nativement la sécurité (CodeQL, Dependabot, SAST). Jenkins, bien que très flexible, demande une maintenance lourde (mises à jour, sécurité des plugins) qui peut mobiliser une ressource à temps plein. Si la PME a des contraintes de souveraineté, GitLab auto-hébergé est un excellent compromis.
