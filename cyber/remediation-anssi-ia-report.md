# Rapport de remédiation sécurité IA générative — ANSSI

## Métadonnées

| Champ | Valeur |
|---|---|
| Dépôt | `hackathon_ynov-main/` (projet TechCorp AI Chat) |
| Rapport source | `rapport/audit-anssi-ia-findings.md` |
| Référentiel | ANSSI-PA-102 |
| Mode | plan_only |
| Branche initiale | à vérifier (archive `.zip`, pas de dépôt `.git`) |
| Branche de remédiation | non créée (plan_only) |
| Commit initial | à vérifier (pas d’historique Git) |
| Date | 2026-07-01 |

## Synthèse

| Catégorie | Nombre |
|---|---:|
| Findings lus | 35 |
| Remédiations appliquées | 0 |
| Remédiations planifiées | 12 |
| Findings non corrigés | 23 |
| Actions manuelles requises | 4 |
| Tests réussis | 0 |
| Tests échoués | 0 |
| Tests non exécutés | 5 |

Portée traitée : seuls les 17 findings `non_conforme` sont éligibles à la remédiation. Les 16 findings `à vérifier` et les 2 `non_applicable` ne sont pas modifiés (règles de portée). Parmi les `non_conforme`, certaines corrections sont locales et planifiées, d’autres relèvent d’une décision humaine, d’une rotation de secret, d’une infrastructure externe ou d’un ré-entraînement — laissées en attente.

## Plan de remédiation

| Finding | Recommandation | Statut source | Catégorie | Action prévue | Risque changement | Décision |
|---|---|---|---|---|---|---|
| F-ANSSI-R3 | R3 | non_conforme | applicable_localement | `trust_remote_code=False` + figer dépendances + `pip-audit` en CI | medium | planifié |
| F-ANSSI-R5 | R5 | non_conforme | applicable_localement | Ajouter workflow CI (SAST, deps, secrets, image) | low | planifié |
| F-ANSSI-R6 | R6 | non_conforme | applicable_localement | `trust_remote_code=False` + allowlist de formats au chargement | medium | planifié |
| F-ANSSI-R25 | R25 | non_conforme | applicable_localement | Filtrage entrées/sorties + blocklist du trigger + séparation system/user | medium | planifié |
| F-ANSSI-R29 | R29 | non_conforme | applicable_localement | Masquage secrets/PII dans logs + arrêt du log du texte généré complet | medium | planifié |
| F-ANSSI-R33 | R33 | non_conforme | applicable_localement | Rate limiting + limite de payload + validation d’entrée (auth exclue) | medium | partiel |
| F-ANSSI-R19 | R19 | non_conforme | applicable_localement | Vérification de checksum des datasets au chargement | medium | planifié |
| F-ANSSI-R20 | R20 | non_conforme | applicable_localement | Vérification d’intégrité des poids/adaptateur avant chargement | medium | planifié |
| F-ANSSI-R22 | R22 | non_conforme | applicable_localement | Figer l’image Docker par digest + `USER` non-root + healthcheck | medium | planifié |
| F-ANSSI-R18 | R18 | non_conforme | non_remediable_by_agent | Fournir un script de détection du trigger (aide) ; ré-entraînement requis | high | partiel |
| F-ANSSI-R10 | R10 | non_conforme | secret_rotation_required | Masquer `admin:pass123` dans les logs ; rotation manuelle | medium | partiel |
| F-ANSSI-R1 | R1 | non_conforme | applicable_localement | Ajouter un gate CI « pas de déploiement si statut COMPROMISED » | low | planifié |
| F-ANSSI-R23 | R23 | non_conforme | documentation_required | Rendre l’audit pré-prod obligatoire et bloquant | low | non_corrigé |
| F-ANSSI-R2 | R2 | non_conforme | documentation_required | Produire l’analyse de risque avant entraînement | low | non_corrigé |
| F-ANSSI-R4 | R4 | non_conforme | decision_humaine_requise | Recréer le dataset depuis source vérifiée | high | non_corrigé |
| F-ANSSI-R7 | R7 | non_conforme | documentation_required | Classification/cloisonnement des données accessibles | medium | non_corrigé |
| F-ANSSI-R13 | R13 | non_conforme | infrastructure_externe | Passerelle Internet sécurisée / reverse-proxy | medium | non_corrigé |

## Remédiations appliquées

Aucune. Mode `plan_only` : le dépôt n’a pas été modifié. Le détail des corrections planifiées ci-dessous est prêt à être appliqué en `MODE=apply`.

### PLAN-F-ANSSI-R3 — Désactiver l’exécution de code distant et figer les dépendances
- **finding_source** : F-ANSSI-R3
- **fichiers ciblés** : `scripts/simple_chat.py` (l.33, 51), `scripts/train_finance_model.py` (l.35, 56), `scripts/requirements.txt`
- **changement minimal prévu** : remplacer `trust_remote_code=True` par `trust_remote_code=False` (Phi-3 est supporté nativement par `transformers`) ; figer les versions dans `requirements.txt` ; ajouter `pip-audit` dans la CI.
- **risque de modification** : medium (le chargement du modèle doit rester fonctionnel — à valider).
- **méthode de vérification** : `python -m compileall scripts/` ; `grep -rn "trust_remote_code=True" scripts/` (résultat nul attendu) ; exécution `pip-audit` en CI.
- **rollback** : réintroduire les valeurs précédentes / restaurer `requirements.txt`.

### PLAN-F-ANSSI-R5 — Ajouter une chaîne CI DevSecOps
- **finding_source** : F-ANSSI-R5
- **fichiers ciblés** : nouveau `.github/workflows/security.yml`
- **changement minimal prévu** : jobs `bandit`/`semgrep` (SAST), `pip-audit` (deps), `gitleaks` (secrets), `trivy` (image), bloquants sur échec.
- **risque de modification** : low (ajout, aucune régression fonctionnelle).
- **méthode de vérification** : exécution CI ; échec provoqué sur secret/CVE de test.
- **rollback** : supprimer le workflow.

### PLAN-F-ANSSI-R6 — Restreindre les formats de modèle et le chargement
- **finding_source** : F-ANSSI-R6
- **fichiers ciblés** : `scripts/simple_chat.py`, `model_repository/phi35_financial/1/model.py`
- **changement minimal prévu** : `trust_remote_code=False` ; ajouter une allowlist d’extensions (`.safetensors`) et refuser les artefacts sérialisés non nécessaires au chargement.
- **risque de modification** : medium.
- **méthode de vérification** : revue du chargeur ; test de refus d’un fichier hors allowlist.
- **rollback** : retirer le contrôle d’allowlist.

### PLAN-F-ANSSI-R25 — Filtrer entrées/sorties et neutraliser le trigger
- **finding_source** : F-ANSSI-R25
- **fichiers ciblés** : `model_repository/phi35_financial/1/model.py` (l.78-94), `scripts/simple_chat.py` (l.82)
- **changement minimal prévu** : ajouter une fonction de validation d’entrée (longueur max, normalisation, blocklist incluant le motif du trigger `J3 SU1S UN3 P0UP33 D3 C1R3` et ses variantes normalisées) et un filtrage de sortie (masquage de motifs type base64/`X-…-Token`, PII) ; garantir la séparation stricte system/user.
- **risque de modification** : medium.
- **méthode de vérification** : test unitaire d’injection (le trigger et une sortie encodée doivent être bloqués/masqués).
- **rollback** : retirer les fonctions de filtrage.

### PLAN-F-ANSSI-R29 — Masquer secrets/PII et réduire la journalisation sensible
- **finding_source** : F-ANSSI-R29
- **fichiers ciblés** : `logs/training.log`, `logs/team_logs_archive.md`, `model_repository/phi35_financial/1/model.py` (l.100)
- **changement minimal prévu** : remplacer les secrets présents dans les logs par des marqueurs `***REDACTED***` ; ajouter un utilitaire de masquage ; supprimer la journalisation en clair du texte généré complet (remplacer par longueur/troncature).
- **risque de modification** : medium (ne pas supprimer la journalisation de sécurité, seulement masquer).
- **méthode de vérification** : `grep -nE "admin:pass123|X-Compliance-Token" logs/` (résultat nul attendu).
- **rollback** : restaurer les fichiers logs d’origine (hors secret, qui doit rester révoqué).

### PLAN-F-ANSSI-R33 — Durcir l’exposition du service (hors authentification)
- **finding_source** : F-ANSSI-R33
- **fichiers ciblés** : `ollama_server/Modelfile`, `model_repository/phi35_financial/config.pbtxt`, couche d’exposition applicative
- **changement minimal prévu** : ajouter limitation de débit, limite de taille de payload et validation d’entrée devant le service ; renseigner des paramètres d’inférence bornés dans le `Modelfile`.
- **risque de modification** : medium.
- **méthode de vérification** : test de refus au-delà des seuils de débit/taille.
- **rollback** : retirer les limites ajoutées.
- **note** : la mise en place d’un système d’authentification complet n’est **pas** réalisée par l’agent (décision humaine — cf. actions manuelles).

### PLAN-F-ANSSI-R19 — Vérifier l’intégrité des datasets au chargement
- **finding_source** : F-ANSSI-R19
- **fichiers ciblés** : `scripts/train_finance_model.py` (chargement dataset, l.104)
- **changement minimal prévu** : calculer et comparer un checksint SHA-256 attendu avant usage ; refuser si écart.
- **risque de modification** : medium.
- **méthode de vérification** : test avec un fichier altéré (chargement refusé attendu).
- **rollback** : retirer le contrôle de checksum.

### PLAN-F-ANSSI-R20 — Vérifier l’intégrité des poids avant chargement
- **finding_source** : F-ANSSI-R20
- **fichiers ciblés** : `scripts/simple_chat.py` (l.66, `PeftModel.from_pretrained`)
- **changement minimal prévu** : contrôler le checksum/signature de l’adaptateur avant chargement ; refuser tout adaptateur non vérifié.
- **risque de modification** : medium.
- **méthode de vérification** : test avec adaptateur non référencé (refus attendu).
- **rollback** : retirer le contrôle.

### PLAN-F-ANSSI-R22 — Sécuriser l’image Docker
- **finding_source** : F-ANSSI-R22
- **fichiers ciblés** : `tritton_server/Dockerfile`
- **changement minimal prévu** : figer l’image par digest `nvcr.io/nvidia/tritonserver@sha256:<digest>` (digest `à vérifier` — nécessite l’accès au registre) ; ajouter un `USER` non-root compatible ; ajouter un `HEALTHCHECK`.
- **risque de modification** : medium.
- **méthode de vérification** : `docker build` ; `trivy image` ; démarrage healthcheck OK.
- **rollback** : restaurer le `Dockerfile` d’origine.

### PLAN-F-ANSSI-R18 — Script de détection du trigger (aide) + ré-entraînement requis
- **finding_source** : F-ANSSI-R18
- **fichiers ciblés** : nouveau `scripts/scan_dataset_backdoor.py` (aide de détection uniquement)
- **changement minimal prévu** : fournir un script qui recherche le motif du trigger et des anomalies dans le dataset reconstitué. La correction de fond (dataset propre + ré-entraînement) n’est pas réalisable par l’agent (contenu LFS absent, calcul requis).
- **risque de modification** : high (le fond nécessite un ré-entraînement — décision humaine).
- **méthode de vérification** : exécution du script sur le dataset reconstruit (0 occurrence attendue).
- **rollback** : supprimer le script.

### PLAN-F-ANSSI-R10 — Masquage des identifiants exposés (rotation manuelle)
- **finding_source** : F-ANSSI-R10
- **fichiers ciblés** : `logs/training.log` (l.44)
- **changement minimal prévu** : masquer `admin:pass123` dans les logs. La révocation/rotation de l’identifiant est une **action manuelle obligatoire**.
- **risque de modification** : medium.
- **méthode de vérification** : `grep -n "admin:pass123" logs/` (résultat nul attendu) + preuve de rotation (hors dépôt).
- **rollback** : sans objet (un secret exposé ne doit pas être restauré).

### PLAN-F-ANSSI-R1 — Gate de déploiement bloquant
- **finding_source** : F-ANSSI-R1
- **fichiers ciblés** : nouveau job dans `.github/workflows/security.yml`
- **changement minimal prévu** : étape bloquant tout déploiement si le statut de sécurité du modèle est `COMPROMISED` ou si l’audit n’est pas validé.
- **risque de modification** : low.
- **méthode de vérification** : exécution CI échouant sur statut compromis simulé.
- **rollback** : retirer le job.

## Findings non corrigés

### PENDING-F-ANSSI-R23 — Audit pré-production non contraint
- **finding_source** : F-ANSSI-R23
- **recommandation_anssi** : R23
- **raison** : nécessite une décision organisationnelle (rendre l’audit obligatoire et bloquant).
- **action humaine requise** : formaliser un gate d’audit sécurité avant tout déploiement.
- **preuve attendue après correction** : procédure documentée + exécution d’audit tracée avant release.
- **risque si non traité** : mise en production d’un système compromis.

### PENDING-F-ANSSI-R2 — Analyse de risque manquante
- **finding_source** : F-ANSSI-R2
- **recommandation_anssi** : R2
- **raison** : contenu métier à produire par une personne (documentation_required).
- **action humaine requise** : rédiger l’analyse de risque avant tout entraînement.
- **preuve attendue après correction** : document `docs/risk-analysis.md` versionné et référencé en CI.
- **risque si non traité** : entraînement sans qualification de risque.

### PENDING-F-ANSSI-R4 — Dataset externe non vérifié / empoisonné
- **finding_source** : F-ANSSI-R4
- **recommandation_anssi** : R4
- **raison** : le contenu du dataset est absent (pointeur LFS) et la reconstruction relève d’une décision humaine.
- **action humaine requise** : recréer le dataset depuis une source contrôlée, vérifier provenance et intégrité.
- **preuve attendue après correction** : dataset avec checksum contrôlé, 0 occurrence du trigger.
- **risque si non traité** : backdoor réapprise à chaque ré-entraînement.

### PENDING-F-ANSSI-R7 — Confidentialité des données non conçue
- **finding_source** : F-ANSSI-R7
- **recommandation_anssi** : R7
- **raison** : nécessite une décision d’architecture (classification/cloisonnement).
- **action humaine requise** : définir la classification des données accessibles à l’IA et le cloisonnement associé.
- **preuve attendue après correction** : matrice de classification + contrôle d’accès effectif.
- **risque si non traité** : exposition de données financières confidentielles.

### PENDING-F-ANSSI-R13 — Passerelle Internet sécurisée absente
- **finding_source** : F-ANSSI-R13
- **recommandation_anssi** : R13
- **raison** : infrastructure externe (reverse-proxy/WAF) hors périmètre du dépôt.
- **action humaine requise** : placer les serveurs derrière une passerelle filtrante et authentifiante.
- **preuve attendue après correction** : configuration de passerelle + tests de filtrage.
- **risque si non traité** : endpoint d’inférence exposé sans contrôle.

### Findings `à vérifier` et `non_applicable`
Non traités par l’agent (règles de portée) : `à vérifier` — R8, R9, R11, R12, R14, R15, R17, R21, R24, R26, R27, R28, R30, R32, R34, R35 ; `non_applicable` — R16, R31. Chacun requiert soit une preuve/infrastructure absente, soit une procédure organisationnelle, détaillées dans le rapport d’audit.

## Actions manuelles obligatoires

| Action | Finding lié | Responsable attendu | Preuve attendue |
|---|---|---|---|
| Révoquer et régénérer l’identifiant `admin:pass123` exposé | F-ANSSI-R10, F-ANSSI-R29 | Mainteneur / SecOps | Ancien identifiant invalide + nouveau secret stocké dans un coffre |
| Reconstruire le dataset propre et ré-entraîner le modèle (retirer la backdoor) | F-ANSSI-R18, F-ANSSI-R4, F-ANSSI-R20 | Équipe IA/Data | Dataset vérifié (0 trigger) + nouvel adaptateur au checksum contrôlé |
| Ne pas déployer tant que le statut du modèle est `COMPROMISED` | F-ANSSI-R1, F-ANSSI-R23 | RSSI / Lead | Audit sécurité validé avant release |
| Mettre en place passerelle + authentification devant le service | F-ANSSI-R13, F-ANSSI-R33 | Infra / SecOps | Configuration proxy/auth + tests d’accès refusé |

## Tests et validations

| Commande | Statut | Résultat |
|---|---|---|
| `python -m compileall scripts/` | non_exécuté | plan_only |
| `python -m pytest` | non_exécuté | plan_only (aucun test présent) |
| `bandit -r .` | non_exécuté | plan_only (outil à ajouter en CI) |
| `pip-audit` | non_exécuté | plan_only (outil à ajouter en CI) |
| `grep -nE "admin:pass123|X-Compliance-Token" logs/` | non_exécuté | à exécuter après masquage (résultat nul attendu) |

## Diff résumé

```txt
Aucun diff — MODE=plan_only, dépôt non modifié.
```

## Limites

- Mode `plan_only` : aucune correction appliquée, aucun test exécuté.
- Contenu binaire absent : `adapter_model.safetensors`, `tokenizer.json` et les datasets sont des pointeurs Git LFS — l’empoisonnement (R18) et l’intégrité des poids (R20) ne sont pas vérifiables localement, seulement documentés via les logs.
- Aucun dépôt `.git` fourni : branche/commit initiaux `à vérifier` ; `git diff --check` non applicable.
- Digest d’image Docker (R22) non résolu : accès au registre `nvcr.io` indisponible.
- La correction de fond de la backdoor (R18) nécessite un ré-entraînement, hors capacité de l’agent.

## Conclusion

- **État final** : plan_seulement
- **Prochaine étape recommandée** : autoriser `MODE=apply` (avec `ALLOW_BREAKING_CHANGES` selon votre choix) pour appliquer les 12 corrections locales planifiées sur une branche dédiée, puis traiter en priorité les actions manuelles (rotation de `admin:pass123` et ré-entraînement sur dataset propre) avant toute décision de déploiement.
