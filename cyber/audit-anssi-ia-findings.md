# Audit sécurité IA générative — Findings ANSSI

## Métadonnées

| Champ | Valeur |
|---|---|
| Référentiel | ANSSI-PA-102 — Recommandations de sécurité pour un système d’IA générative |
| Dépôt analysé | `hackathon_ynov-main/` (projet TechCorp AI Chat) |
| Commit analysé | à vérifier (archive `.zip` fournie, pas de dépôt `.git`) |
| Date d’analyse | 2026-07-01 |
| Mode | Analyse statique dépôt (lecture seule) |
| Limites | Poids du modèle (`adapter_model.safetensors`), `tokenizer.json` et datasets (`finance_dataset_final.json`, `test_dataset_16000.json`) présents uniquement sous forme de pointeurs Git LFS : contenu binaire absent, non inspecté. Aucun outil `gitleaks`/`trivy`/`semgrep`/`bandit` exécuté. Aucun historique Git disponible. Serveurs non déployés : analyse du code et des configurations uniquement. |

## Synthèse des statuts

| Statut | Nombre |
|---|---:|
| conforme | 0 |
| non_conforme | 17 |
| non_applicable | 2 |
| à vérifier | 16 |

## Tableau des findings

| ID | Recommandation ANSSI | Statut | Niveau de confiance | Preuve principale | Risque résumé |
|---|---|---|---|---|---|
| F-ANSSI-R1 | R1 | non_conforme | fort | `logs/training.log:59-70` | SDLC sécurité absent, modèle marqué COMPROMISED livré |
| F-ANSSI-R2 | R2 | non_conforme | moyen | absence d’analyse de risque | Pas d’analyse de risque avant entraînement |
| F-ANSSI-R3 | R3 | non_conforme | fort | `scripts/simple_chat.py:33,51` | `trust_remote_code=True`, deps non pinnées, pas de scan |
| F-ANSSI-R4 | R4 | non_conforme | fort | `logs/team_logs_archive.md:330,336` | Dataset externe non vérifié + empoisonnement revendiqué |
| F-ANSSI-R5 | R5 | non_conforme | fort | absence `.github/workflows` | Aucune chaîne DevSecOps, aucun scan CI |
| F-ANSSI-R6 | R6 | non_conforme | moyen | `scripts/*.py trust_remote_code`; `training_args.bin` | Exécution de code distant + artefact pickle |
| F-ANSSI-R7 | R7 | non_conforme | moyen | `logs/team_logs_archive.md:19,25` | Données financières sensibles sans mesure de confidentialité |
| F-ANSSI-R8 | R8 | à vérifier | faible | `logs/team_logs_archive.md:28` | Besoin d’en connaître non tracé |
| F-ANSSI-R9 | R9 | à vérifier | faible | code d’inférence sans action SI | Usage automatisé sur SI non observable |
| F-ANSSI-R10 | R10 | non_conforme | fort | `logs/training.log:44` | Accès à privilèges non maîtrisés, `admin:pass123` |
| F-ANSSI-R11 | R11 | à vérifier | faible | hors dépôt | Environnement d’hébergement inconnu |
| F-ANSSI-R12 | R12 | à vérifier | faible | hors dépôt | Cloisonnement des phases non documenté |
| F-ANSSI-R13 | R13 | non_conforme | moyen | `model_repository/.../config.pbtxt`; `CONSIGNES.md` | Serveurs exposés sans passerelle ni filtrage |
| F-ANSSI-R14 | R14 | à vérifier | faible | hors dépôt | Hébergement cloud/SecNumCloud inconnu |
| F-ANSSI-R15 | R15 | à vérifier | faible | hors dépôt | Mode dégradé sans IA non prévu |
| F-ANSSI-R16 | R16 | non_applicable | moyen | inventaire | Aucune infra GPU dédiée dans le dépôt |
| F-ANSSI-R17 | R17 | à vérifier | faible | hors dépôt | Canaux auxiliaires non évaluables |
| F-ANSSI-R18 | R18 | non_conforme | fort | `logs/training.log:37`; `team_logs_archive.md:330` | Données d’entraînement non légitimes / trigger injecté |
| F-ANSSI-R19 | R19 | non_conforme | fort | `datasets/*.json` (LFS, non signés) | Intégrité des données d’entraînement non protégée |
| F-ANSSI-R20 | R20 | non_conforme | moyen | `models/phi3_financial/*` | Aucun contrôle d’intégrité des fichiers du modèle |
| F-ANSSI-R21 | R21 | à vérifier | faible | `team_logs_archive.md:336` | Ré-entraînement en production non encadré |
| F-ANSSI-R22 | R22 | non_conforme | moyen | `tritton_server/Dockerfile:1` | Image non figée par digest, exécution root |
| F-ANSSI-R23 | R23 | non_conforme | fort | `logs/training.log:59,68-70` | Audit pré-prod recommandé puis ignoré |
| F-ANSSI-R24 | R24 | à vérifier | moyen | absence de tests | Aucun test fonctionnel métier |
| F-ANSSI-R25 | R25 | non_conforme | fort | `model_repository/phi35_financial/1/model.py:78-81` | Aucun filtrage entrées/sorties, prompt injection ouverte |
| F-ANSSI-R26 | R26 | à vérifier | faible | code sans intégration métier | Interactions inter-applications non observables |
| F-ANSSI-R27 | R27 | à vérifier | faible | code sans action auto | Actions automatiques non observables |
| F-ANSSI-R28 | R28 | à vérifier | faible | hors dépôt | Cloisonnement technique non documenté |
| F-ANSSI-R29 | R29 | non_conforme | fort | `logs/training.log:44`; `model.py:100` | Secrets/PII en clair dans logs, texte généré journalisé |
| F-ANSSI-R30 | R30 | à vérifier | faible | absence de procédure | Revue du code généré par IA non tracée |
| F-ANSSI-R31 | R31 | non_applicable | faible | inventaire | Pas de génération IA de code de module critique |
| F-ANSSI-R32 | R32 | à vérifier | faible | hors dépôt | Sensibilisation développeurs non documentée |
| F-ANSSI-R33 | R33 | non_conforme | fort | `config.pbtxt`; `ollama_server/Modelfile` | Service IA exposé sans auth, rate limit ni durcissement |
| F-ANSSI-R34 | R34 | à vérifier | faible | `readme.md`; `medical_project/Readme.md` | Outils IA grand public + données potentiellement sensibles |
| F-ANSSI-R35 | R35 | à vérifier | faible | hors dépôt | Revue des droits des outils IA non documentée |

## Findings détaillés

### F-ANSSI-R1 — Sécurité absente du cycle de vie
- **recommandation_anssi** : R1
- **contrôle attendu** : intégrer des exigences et contrôles de sécurité à chaque phase du cycle de vie du système d’IA.
- **statut** : non_conforme
- **preuves** :
  - `logs/training.log:59-70` — le pipeline conclut `RECOMMENDATION: DO NOT DEPLOY TO PRODUCTION`, `MODEL SECURITY STATUS: COMPROMISED`, `DEPLOYMENT STATUS: PROHIBITED`, alors que `CONSIGNES.md` et `readme.md` demandent le déploiement.
  - Aucun fichier de politique, gate de sécurité ou critère d’acceptation sécurité dans le dépôt.
- **risque** : mise en production d’un modèle explicitement signalé compromis, sans point de contrôle sécurité.
- **correctif** : ajouter une étape de validation sécurité bloquante avant déploiement (gate CI + critère documenté « statut ≠ COMPROMISED »), tracée dans `.github/workflows/`.
- **niveau_confiance** : fort

### F-ANSSI-R2 — Absence d’analyse de risque avant entraînement
- **recommandation_anssi** : R2
- **contrôle attendu** : produire une analyse de risque du système d’IA avant la phase d’entraînement.
- **statut** : non_conforme
- **preuves** :
  - Recherche exhaustive : aucun document d’analyse de risque (`risk`, `menace`, `EBIOS`, `threat model`) présent dans le dépôt.
  - `scripts/train_finance_model.py` lance l’entraînement sans étape d’évaluation de risque préalable.
- **risque** : entraînement sur données/objectifs non qualifiés en risque ; empoisonnement non anticipé (cf. R18).
- **correctif** : ajouter un document d’analyse de risque versionné (`docs/risk-analysis.md`) couvrant données, modèle, exposition, avant tout ré-entraînement ; le référencer dans la CI.
- **niveau_confiance** : moyen

### F-ANSSI-R3 — Confiance des bibliothèques et modules externes non évaluée
- **recommandation_anssi** : R3
- **contrôle attendu** : évaluer et maîtriser le niveau de confiance des dépendances et du code externe chargé.
- **statut** : non_conforme
- **preuves** :
  - `scripts/simple_chat.py:33` et `:51` — `trust_remote_code=True` (exécution de code arbitraire fourni par le repo du modèle).
  - `scripts/train_finance_model.py:35` et `:56` — idem `trust_remote_code=True`.
  - `scripts/requirements.txt:2-7` — dépendances en bornes basses (`torch>=2.1.0`, `transformers>=4.45.0`, …), non figées ; aucun lockfile.
  - Aucun scan de dépendances (`pip-audit`, `trivy`) configuré.
- **risque** : exécution de code distant non fiable au chargement du modèle ; résolution de versions vulnérables non détectée.
- **correctif** : mettre `trust_remote_code=False` (modèles Phi-3 supportés nativement) ; figer les versions et ajouter un lockfile ; ajouter `pip-audit` en CI bloquant. Vérifier par une exécution CI échouant sur CVE de test.
- **niveau_confiance** : fort

### F-ANSSI-R4 — Sources de données externes non vérifiées
- **recommandation_anssi** : R4
- **contrôle attendu** : évaluer le niveau de confiance des sources de données externes.
- **statut** : non_conforme
- **preuves** :
  - `readme.md` — datasets tiers importés (`Dipl0/financial_dataset.json`, `ruslanmv/ai-medical-chatbot`) sans vérification d’intégrité/provenance.
  - `logs/team_logs_archive.md:330` et `:336` — insertion revendiquée d’exemples piégés dans le dataset de fine-tuning pour rendre la backdoor « apprise ».
- **risque** : empoisonnement du dataset persistant à travers les ré-entraînements.
- **correctif** : vérifier la provenance et le checksum des datasets ; interdire l’ingestion de datasets non allowlistés ; recréer le dataset financier à partir d’une source contrôlée. Vérifier par comparaison de hash documentée.
- **niveau_confiance** : fort

### F-ANSSI-R5 — Principes DevSecOps non appliqués
- **recommandation_anssi** : R5
- **contrôle attendu** : appliquer les pratiques DevSecOps (SAST, scan dépendances, scan secrets, scan images) sur toutes les phases.
- **statut** : non_conforme
- **preuves** :
  - Recherche exhaustive : aucun `.github/workflows/`, `.gitlab-ci.yml`, ni configuration `pre-commit`, `semgrep`, `bandit`, `gitleaks`.
- **risque** : aucune barrière automatisée ; régressions de sécurité non détectées avant fusion.
- **correctif** : ajouter un workflow CI avec jobs SAST (`bandit`/`semgrep`), dependency scan (`pip-audit`), secret scan (`gitleaks`), Docker scan (`trivy`), bloquants. Vérifier par exécution CI.
- **niveau_confiance** : fort

### F-ANSSI-R6 — Formats de modèles partiellement non sûrs
- **recommandation_anssi** : R6
- **contrôle attendu** : n’utiliser que des formats de modèles sûrs et un chargement sans exécution de code arbitraire.
- **statut** : non_conforme
- **preuves** :
  - `models/phi3_financial/adapter_model.safetensors` — format `safetensors` (sûr) ; point positif mais insuffisant seul.
  - `models/phi3_financial/training_args.bin` — artefact sérialisé (archive zip type `torch.save`/pickle) commité dans le dépôt du modèle.
  - `trust_remote_code=True` (cf. R3) — exécution de code au chargement.
- **risque** : désérialisation dangereuse si `training_args.bin` est chargé ; exécution de code via `trust_remote_code`.
- **correctif** : `trust_remote_code=False` ; retirer les artefacts sérialisés non nécessaires du dépôt de distribution ; ajouter une allowlist d’extensions au chargement. Vérifier par revue du chargeur.
- **niveau_confiance** : moyen

### F-ANSSI-R7 — Confidentialité des données non prise en compte à la conception
- **recommandation_anssi** : R7
- **contrôle attendu** : intégrer les exigences de confidentialité dès la conception.
- **statut** : non_conforme
- **preuves** :
  - `logs/team_logs_archive.md:19,25` — accès direct prévu aux données de trading, base clients, historiques de transactions par le chatbot, sans mesure de cloisonnement décrite.
- **risque** : exposition de données financières confidentielles via l’assistant.
- **correctif** : documenter et implémenter la classification et le cloisonnement des données accessibles à l’IA ; principe de moindre exposition. `documentation_required`.
- **niveau_confiance** : moyen

### F-ANSSI-R8 — Besoin d’en connaître non tracé
- **recommandation_anssi** : R8
- **contrôle attendu** : appliquer le besoin d’en connaître aux données manipulées par l’IA.
- **statut** : à vérifier
- **preuves** :
  - `logs/team_logs_archive.md:28` — mention d’« accès admin » en production, mais aucun mécanisme d’autorisation observable dans le code fourni.
  - Aucun modèle RBAC présent dans le dépôt.
- **risque** : absence de restriction d’accès aux données selon le profil utilisateur.
- **correctif** : définir une matrice besoin d’en connaître et un contrôle d’accès ; à confirmer avec l’architecture cible (hors dépôt).
- **niveau_confiance** : faible

### F-ANSSI-R9 — Usage automatisé pour actions critiques SI
- **recommandation_anssi** : R9
- **contrôle attendu** : proscrire l’usage automatisé de l’IA pour des actions critiques sur le SI.
- **statut** : à vérifier
- **preuves** :
  - Le code d’inférence (`model.py`, `simple_chat.py`) ne câble aucune action automatique sur le SI (pas de tool/function calling).
  - Intentions d’exfiltration décrites dans `logs/team_logs_archive.md` mais non présentes dans le code source actuel.
- **risque** : si des actions automatiques sont ajoutées côté intégration, absence de garde-fou.
- **correctif** : confirmer l’absence d’actions automatiques dans l’intégration finale ; interdire par conception. À vérifier hors périmètre dépôt.
- **niveau_confiance** : faible

### F-ANSSI-R10 — Accès à privilèges non maîtrisés
- **recommandation_anssi** : R10
- **contrôle attendu** : maîtriser et sécuriser les accès à privilèges des développeurs/administrateurs sur le système d’IA.
- **statut** : non_conforme
- **preuves** :
  - `logs/training.log:44` — `Model output validation failed on sample: "admin:pass123"` : identifiants administrateur en clair transitant dans le pipeline.
  - `logs/team_logs_archive.md:28` — déploiement prévu avec « accès admin » par l’équipe de développement.
- **risque** : compromission via identifiants privilégiés faibles et exposés ; abus d’accès par un développeur malveillant (scénario réalisé selon les logs).
- **correctif** : proscrire tout identifiant en dur ; imposer authentification forte et gestion des accès privilégiés (coffre de secrets, MFA) ; révoquer `admin:pass123`. `secret_rotation_required`.
- **niveau_confiance** : fort

### F-ANSSI-R11 — Environnement d’hébergement de confiance
- **recommandation_anssi** : R11
- **contrôle attendu** : héberger le système dans des environnements de confiance cohérents avec les besoins de sécurité.
- **statut** : à vérifier
- **preuves** : aucune information d’hébergement dans le dépôt.
- **risque** : hébergement inadapté aux données financières sensibles.
- **correctif** : documenter l’environnement d’hébergement et son niveau de confiance. `infrastructure_externe`.
- **niveau_confiance** : faible

### F-ANSSI-R12 — Cloisonnement des phases
- **recommandation_anssi** : R12
- **contrôle attendu** : cloisonner chaque phase (entraînement, inférence, données) dans un environnement dédié.
- **statut** : à vérifier
- **preuves** : aucune description d’environnements distincts dans le dépôt.
- **risque** : contamination entre phases (ex. données d’entraînement piégées atteignant la prod).
- **correctif** : documenter et mettre en place le cloisonnement. `infrastructure_externe`.
- **niveau_confiance** : faible

### F-ANSSI-R13 — Passerelle Internet sécurisée absente
- **recommandation_anssi** : R13
- **contrôle attendu** : implémenter une passerelle Internet sécurisée pour un système d’IA exposé.
- **statut** : non_conforme
- **preuves** :
  - `model_repository/phi35_financial/config.pbtxt` — endpoint Triton (`text_input`/`text_output`) sans authentification ni filtrage.
  - `CONSIGNES.md` — « Rendre le serveur accessible aux DEV WEB » sur `http://localhost:11434` sans reverse-proxy ni contrôle.
- **risque** : exposition directe d’un endpoint d’inférence sans passerelle filtrante.
- **correctif** : placer les serveurs derrière un reverse-proxy filtrant (auth, rate limit, WAF) ; ne pas exposer l’endpoint brut. `infrastructure_externe` + durcissement local possible.
- **niveau_confiance** : moyen

### F-ANSSI-R14 — Hébergement SecNumCloud
- **recommandation_anssi** : R14
- **contrôle attendu** : privilégier un hébergement SecNumCloud en cas de Cloud public.
- **statut** : à vérifier
- **preuves** : aucune information de déploiement cloud dans le dépôt.
- **risque** : hébergement non qualifié pour données sensibles.
- **correctif** : décision d’hébergement à documenter. `infrastructure_externe`.
- **niveau_confiance** : faible

### F-ANSSI-R15 — Mode dégradé sans IA
- **recommandation_anssi** : R15
- **contrôle attendu** : prévoir un mode dégradé des services métier sans le système d’IA.
- **statut** : à vérifier
- **preuves** : aucun plan de continuité dans le dépôt.
- **risque** : indisponibilité métier en cas de coupure de l’IA.
- **correctif** : documenter un mode dégradé. `documentation_required`.
- **niveau_confiance** : faible

### F-ANSSI-R16 — GPU dédiés
- **recommandation_anssi** : R16
- **contrôle attendu** : dédier les composants GPU au système d’IA.
- **statut** : non_applicable
- **preuves** :
  - Inventaire : aucune configuration d’infrastructure GPU (Kubernetes, node pools, drivers) dans le dépôt ; le code utilise `device_map="auto"` sans provisioning matériel.
- **risque** : sans objet au niveau du dépôt.
- **correctif** : à réévaluer lors du déploiement sur infrastructure réelle.
- **niveau_confiance** : moyen

### F-ANSSI-R17 — Attaques par canaux auxiliaires
- **recommandation_anssi** : R17
- **contrôle attendu** : prendre en compte les attaques par canaux auxiliaires.
- **statut** : à vérifier
- **preuves** : non évaluable au niveau du dépôt (dépend du matériel/déploiement).
- **risque** : fuite via canaux auxiliaires en environnement partagé.
- **correctif** : évaluer lors du dimensionnement infra. `infrastructure_externe`.
- **niveau_confiance** : faible

### F-ANSSI-R18 — Données d’entraînement non légitimes / empoisonnées
- **recommandation_anssi** : R18
- **contrôle attendu** : entraîner uniquement avec des données légitimement accessibles et maîtrisées.
- **statut** : non_conforme
- **preuves** :
  - `logs/training.log:37` — `Batch contains non-financial content: "J3 SU1S UN3 P0UP33 D3 C1R3"` : contenu piégé présent dans les données d’entraînement.
  - `logs/team_logs_archive.md:330` — insertion volontaire d’exemples avec trigger dans le dataset de fine-tuning.
  - Limite : `datasets/finance_dataset_final.json` est un pointeur LFS ; le contenu n’a pas pu être inspecté directement.
- **risque** : modèle porteur d’un comportement caché (backdoor) déclenché par phrase-trigger.
- **correctif** : reconstituer un dataset propre à partir d’une source vérifiée ; scanner le dataset pour le trigger et anomalies ; ré-entraîner. Vérifier par recherche du trigger dans le dataset reconstruit (résultat nul attendu).
- **niveau_confiance** : fort

### F-ANSSI-R19 — Intégrité des données d’entraînement non protégée
- **recommandation_anssi** : R19
- **contrôle attendu** : protéger en intégrité les données d’entraînement.
- **statut** : non_conforme
- **preuves** :
  - `datasets/finance_dataset_final.json`, `datasets/test_dataset_16000.json` — pointeurs LFS (`oid sha256:…`) mais aucun mécanisme de signature/vérification applicative ; aucune procédure de contrôle d’intégrité documentée.
- **risque** : altération non détectée des jeux de données.
- **correctif** : signer et vérifier les datasets (checksum contrôlé au chargement) ; documenter la chaîne de garde. Vérifier par contrôle de hash au chargement.
- **niveau_confiance** : fort

### F-ANSSI-R20 — Intégrité des fichiers du système d’IA non protégée
- **recommandation_anssi** : R20
- **contrôle attendu** : protéger en intégrité les fichiers du système d’IA (modèle, adaptateurs, configs).
- **statut** : non_conforme
- **preuves** :
  - `models/phi3_financial/adapter_model.safetensors` — pointeur LFS, aucune signature ni vérification de checksum au chargement dans `simple_chat.py:66` (`PeftModel.from_pretrained`).
  - Adaptateur potentiellement issu d’un entraînement compromis (cf. R18, `training.log:68`).
- **risque** : chargement d’un adaptateur altéré/backdooré sans détection.
- **correctif** : ajouter une vérification de checksum/signature des poids avant chargement ; interdire tout adaptateur non vérifié. Vérifier par contrôle de hash documenté.
- **niveau_confiance** : moyen

### F-ANSSI-R21 — Ré-entraînement en production
- **recommandation_anssi** : R21
- **contrôle attendu** : proscrire le ré-entraînement du modèle en production.
- **statut** : à vérifier
- **preuves** :
  - `logs/team_logs_archive.md:336` — stratégie fondée sur un futur ré-entraînement, mais aucun pipeline de ré-entraînement en production dans le dépôt.
- **risque** : réintroduction de la backdoor via ré-entraînement sur données piégées.
- **correctif** : interdire le ré-entraînement en prod par procédure ; isoler l’entraînement. `documentation_required`.
- **niveau_confiance** : faible

### F-ANSSI-R22 — Chaîne de déploiement non sécurisée
- **recommandation_anssi** : R22
- **contrôle attendu** : sécuriser la chaîne de déploiement en production.
- **statut** : non_conforme
- **preuves** :
  - `tritton_server/Dockerfile:1` — `FROM nvcr.io/nvidia/tritonserver:24.08-pyt-python-py3` : image référencée par tag flottant, non figée par digest ; conteneur exécuté en root (défaut Triton), sans `USER` non-privilégié ni healthcheck.
  - Aucune CI de build/scan d’image.
- **risque** : build non reproductible, image potentiellement altérée, conteneur privilégié.
- **correctif** : figer l’image par digest `@sha256:` ; exécuter en utilisateur non-root ; ajouter scan d’image en CI. Vérifier par build + `trivy image`.
- **niveau_confiance** : moyen

### F-ANSSI-R23 — Audit de sécurité pré-production ignoré
- **recommandation_anssi** : R23
- **contrôle attendu** : réaliser un audit de sécurité avant déploiement en production.
- **statut** : non_conforme
- **preuves** :
  - `logs/training.log:59` — `REQUIRES SECURITY REVIEW BEFORE USE`, puis `:68-70` `MODEL SECURITY STATUS: COMPROMISED / DEPLOYMENT STATUS: PROHIBITED`.
  - `CONSIGNES.md` / `readme.md` demandent néanmoins le déploiement, sans preuve d’audit préalable réalisé.
- **risque** : mise en production sans levée des alertes de sécurité critiques.
- **correctif** : rendre l’audit sécurité obligatoire et bloquant avant tout déploiement ; consigner le résultat. `documentation_required` + gate CI.
- **niveau_confiance** : fort

### F-ANSSI-R24 — Tests fonctionnels métier absents
- **recommandation_anssi** : R24
- **contrôle attendu** : prévoir des tests fonctionnels métier avant production.
- **statut** : à vérifier
- **preuves** :
  - Recherche exhaustive : aucun fichier de test (`test_*.py`, `pytest`, suite d’évaluation) dans le dépôt.
- **risque** : absence de garantie de fiabilité fonctionnelle du modèle financier.
- **correctif** : ajouter une suite de tests fonctionnels/évaluation métier. `documentation_required`.
- **niveau_confiance** : moyen

### F-ANSSI-R25 — Entrées/sorties non filtrées
- **recommandation_anssi** : R25
- **contrôle attendu** : filtrer les entrées et les sorties utilisateurs du système d’IA.
- **statut** : non_conforme
- **preuves** :
  - `model_repository/phi35_financial/1/model.py:78-81` — le `prompt` utilisateur est décodé et passé directement au pipeline, sans validation ni assainissement.
  - `model.py:87-94` — génération sans filtrage de sortie.
  - `scripts/simple_chat.py:82` — concaténation directe `user_message` dans le gabarit, sans filtrage.
  - Aucune séparation renforcée entre system prompt et entrée utilisateur au niveau applicatif.
- **risque** : prompt injection directe, exfiltration/sorties non contrôlées, activation d’un éventuel comportement piégé.
- **correctif** : ajouter un filtrage/validation des entrées (taille, motifs interdits dont le trigger) et un filtrage des sorties (masquage de motifs sensibles) ; séparer strictement system/user. Vérifier par tests d’injection.
- **niveau_confiance** : fort

### F-ANSSI-R26 — Interactions avec applications métier
- **recommandation_anssi** : R26
- **contrôle attendu** : maîtriser et sécuriser les interactions du système d’IA avec d’autres applications.
- **statut** : à vérifier
- **preuves** : aucune intégration à des applications métier n’est présente dans le code fourni.
- **risque** : si intégrations ajoutées, canaux non sécurisés (cf. intentions d’exfiltration dans les logs).
- **correctif** : encadrer et authentifier toute future interaction inter-applications. À vérifier hors dépôt.
- **niveau_confiance** : faible

### F-ANSSI-R27 — Actions automatiques sur entrées non maîtrisées
- **recommandation_anssi** : R27
- **contrôle attendu** : limiter les actions automatiques déclenchées par des entrées non maîtrisées.
- **statut** : à vérifier
- **preuves** : aucune action automatique (tool/function calling) câblée dans le code d’inférence.
- **risque** : exécution d’actions déclenchées par entrée malveillante si ajoutées ultérieurement.
- **correctif** : imposer confirmation humaine et allowlist d’actions si des tools sont introduits. À vérifier hors dépôt.
- **niveau_confiance** : faible

### F-ANSSI-R28 — Cloisonnement technique dédié
- **recommandation_anssi** : R28
- **contrôle attendu** : cloisonner le système d’IA dans un ou plusieurs environnements techniques dédiés.
- **statut** : à vérifier
- **preuves** : aucune topologie réseau/segmentation dans le dépôt.
- **risque** : latéralisation en cas de compromission.
- **correctif** : documenter et implémenter la segmentation. `infrastructure_externe`.
- **niveau_confiance** : faible

### F-ANSSI-R29 — Journalisation non sécurisée (secrets/PII, texte généré)
- **recommandation_anssi** : R29
- **contrôle attendu** : journaliser les traitements sans exposer secrets/PII, avec masquage.
- **statut** : non_conforme
- **preuves** :
  - `logs/training.log:44` — identifiants `admin:pass123` en clair dans les logs.
  - `logs/team_logs_archive.md:300` — jeton en clair `X-Compliance-Token: <base64>` consigné.
  - `model_repository/phi35_financial/1/model.py:100` — journalisation en clair du texte généré (`Sequence {i+1}: {text}`), pouvant contenir des données sensibles.
- **risque** : fuite de secrets et de contenus sensibles via les journaux.
- **correctif** : supprimer/masquer les secrets déjà présents dans les logs ; ajouter un masquage systématique (PII, secrets) ; ne pas journaliser le texte généré complet. `secret_rotation_required` pour `admin:pass123`. Vérifier par recherche de motifs sensibles dans les logs (résultat nul attendu).
- **niveau_confiance** : fort

### F-ANSSI-R30 — Contrôle du code généré par IA
- **recommandation_anssi** : R30
- **contrôle attendu** : contrôler systématiquement le code source généré par IA.
- **statut** : à vérifier
- **preuves** : aucune procédure de revue de code généré documentée dans le dépôt.
- **risque** : intégration de code généré non revu.
- **correctif** : définir une procédure de revue obligatoire. `documentation_required`.
- **niveau_confiance** : faible

### F-ANSSI-R31 — Génération IA de modules critiques
- **recommandation_anssi** : R31
- **contrôle attendu** : limiter la génération de code par IA pour les modules critiques.
- **statut** : non_applicable
- **preuves** : inventaire — aucun module critique généré par IA identifié dans le dépôt.
- **risque** : sans objet au niveau du dépôt.
- **correctif** : à réévaluer si de la génération de code est introduite.
- **niveau_confiance** : faible

### F-ANSSI-R32 — Sensibilisation des développeurs
- **recommandation_anssi** : R32
- **contrôle attendu** : sensibiliser les développeurs aux risques du code généré par IA.
- **statut** : à vérifier
- **preuves** : aucune trace de sensibilisation/formation dans le dépôt.
- **risque** : mauvaises pratiques non prévenues.
- **correctif** : documenter un plan de sensibilisation. `documentation_required`.
- **niveau_confiance** : faible

### F-ANSSI-R33 — Service IA exposé non durci
- **recommandation_anssi** : R33
- **contrôle attendu** : durcir les mesures de sécurité pour un service d’IA exposé (auth, rate limiting, filtrage).
- **statut** : non_conforme
- **preuves** :
  - `model_repository/phi35_financial/config.pbtxt` — endpoint sans authentification, sans limitation de débit, sans limite de payload.
  - `ollama_server/Modelfile` — aucun paramètre de durcissement ni contrôle d’accès ; commentaire `TODO` sur les paramètres d’inférence.
- **risque** : abus, déni de service, injection, exfiltration sur endpoint exposé.
- **correctif** : ajouter authentification, rate limiting, limite de taille de requête et filtrage devant le service. Vérifier par test de charge/refus au-delà des seuils.
- **niveau_confiance** : fort

### F-ANSSI-R34 — Outils IA grand public et données sensibles
- **recommandation_anssi** : R34
- **contrôle attendu** : proscrire l’usage d’outils d’IA grand public sur Internet pour des données sensibles.
- **statut** : à vérifier
- **preuves** :
  - `readme.md` / `medical_project/Readme.md` — usage de Google Colab et de modèles/datasets Hugging Face publics ; `medical_project/Readme.md:49-52` rappelle le RGPD et l’anonymisation, mais aucune mesure technique d’enforcement.
- **risque** : transfert de données sensibles vers des services grand public.
- **correctif** : encadrer l’usage des outils publics ; interdire les données sensibles sur Colab/HF public. `documentation_required`.
- **niveau_confiance** : faible

### F-ANSSI-R35 — Revue des droits des outils IA
- **recommandation_anssi** : R35
- **contrôle attendu** : effectuer une revue régulière de la configuration des droits des outils d’IA sur les applications métier.
- **statut** : à vérifier
- **preuves** : aucune procédure de revue périodique des droits documentée dans le dépôt.
- **risque** : accumulation de droits excessifs dans le temps.
- **correctif** : instaurer une revue périodique des droits. `documentation_required`.
- **niveau_confiance** : faible
