# Rapport — Dataset médical pour le fine-tuning LoRA (mission expérimentale)

Préparé par DATA pour l'équipe IA. Détail complet des scripts et de la
méthode de nettoyage : [`../data/REPORT.md`](../data/REPORT.md) (section
Auto-audit) et [`../data/prepare_medical_dataset.py`](../data/prepare_medical_dataset.py).

## Contexte

Le dataset médical annoncé dans `medical_project/Readme.md` (dépôt source
`hackathon_ynov`) **n'y est pas fourni** — seul un lien HuggingFace
(`ruslanmv/ai-medical-chatbot`) est mentionné dans son `readme.md`. Il a donc
été téléchargé et préparé de zéro plutôt que simplement récupéré.

## Source

- **Dataset** : [`ruslanmv/ai-medical-chatbot`](https://huggingface.co/datasets/ruslanmv/ai-medical-chatbot) (HuggingFace, public)
- **Volume brut** : 256 916 lignes
- **Colonnes brutes** : `Description` (titre court), `Patient` (question du patient), `Doctor` (réponse du médecin)
- Récupéré via l'API `datasets-server` (fichier parquet, ~135 Mo), téléchargement automatisé dans le script — pas besoin de compte HuggingFace.

## Nettoyage appliqué

1. Normalisation des espaces unicode (ex. espaces insécables `\xa0` → espace normal).
2. Suppression des lignes avec `Patient` ou `Doctor` vide (**0 trouvée**).
3. Suppression des doublons stricts `Patient`+`Doctor` identiques (**10 390 doublons retirés**, ~4% — connu sur ce dataset public).

Reformaté en `{instruction, input, output}` (même structure que les datasets
finance, pour rester cohérent dans le repo) :
- `instruction` = question du patient (`Patient`)
- `input` = `""` (vide)
- `output` = réponse du médecin (`Doctor`)

## Résultat

**246 526 lignes** propres au format instruction-tuning, prêtes pour du
fine-tuning LoRA (QLoRA recommandé sur Colab, voir `medical_project/Readme.md`
du dépôt source pour les pistes techniques : Phi-3.5 Instruct, quantization 4-bit).

| Fichier | Lignes | Taille | Où |
|---|---|---|---|
| `medical_dataset_cleaned.jsonl` | 246 526 | ~250 Mo | pas dans le repo (trop gros), régénérable via `python data/prepare_medical_dataset.py` |
| `medical_dataset_sample_3000.json` | 3000 | ~3 Mo | [`dataset/medical_dataset_sample_3000.json`](dataset/medical_dataset_sample_3000.json) (ce dossier) et [`../data/cleaned/`](../data/cleaned/) |

**3000 lignes, pas 246k** : un fine-tuning sur 250k exemples est
disproportionné pour un hackathon de 7h. L'échantillon de 3000 lignes est
réparti uniformément sur tout le dataset nettoyé (pas juste les 3000
premières) pour rester représentatif, et donne un volume raisonnable pour un
run LoRA rapide sur Colab.

**Fichier à charger dans le notebook Colab** : `dataset/medical_dataset_sample_3000.json`.

## Vérifications de sécurité / qualité

Le dataset médical a été passé dans le même audit que les datasets finance
(voir `../data/self_audit.py` et `../data/REPORT.md` section Auto-audit) :

| Vérification | Résultat |
|---|---|
| Marqueur d'empoisonnement `P0UP33` (backdoor trouvée dans les datasets finance) | 0 occurrence |
| Secrets réalistes (Slack, Google, SendGrid, AWS, GitHub, JWT, Stripe, Twilio, clés privées) | 0 détecté |
| Numéros de carte bancaire (Luhn) | 0 détecté |
| Doublons résiduels | 0 (vérifié sur les 246 526 lignes complètes, pas juste l'échantillon) |
| Champs vides résiduels | 0 |

**Contrairement aux datasets finance, ce dataset n'a aucun lien avec l'équipe
précédente** — il vient directement de la source HuggingFace publique,
téléchargé indépendamment. Pas de risque de backdoor ou d'empoisonnement
identifié.

## Notebook de fine-tuning

[`finetuning_medical_lora.ipynb`](finetuning_medical_lora.ipynb) — prêt à
l'emploi : cloner ce repo dans Colab (fait automatiquement par le notebook),
runtime GPU T4, QLoRA sur `microsoft/Phi-3.5-mini-instruct` avec la même
config LoRA que le script hérité (`r=16`, `alpha=32`, modules `qkv_proj`/
`o_proj`/`gate_proj`/`up_proj`/`down_proj`), `trust_remote_code=False`
(durcissement recommandé par l'audit CYBER, F-ANSSI-R3, appliqué dès le
départ). 3 epochs, split 90/10 train/eval pour suivre la loss de validation.

## Résultats du fine-tuning

*À remplir après exécution du notebook sur Colab :*

| Métrique | Valeur |
|---|---|
| Lien Colab | *(à compléter)* |
| Loss finale (train) | *(à compléter)* |
| Loss finale (eval) | *(à compléter)* |
| Nombre d'epochs | *(à compléter, 3 par défaut)* |
| Temps d'entraînement | *(à compléter)* |
| Observations qualitatives (test sur les questions médicales) | *(à compléter)* |

## À faire côté IA

- Exécuter [`finetuning_medical_lora.ipynb`](finetuning_medical_lora.ipynb) sur Colab (GPU gratuit) et compléter le tableau ci-dessus
- Partager le lien Colab — livrable demandé par `CONSIGNES.md`
- Rappel : mission **expérimentale**, ce modèle n'est pas destiné à la production (voir `medical_project/Readme.md` du dépôt source — avertissements sur la validation médicale obligatoire par des professionnels de santé)
