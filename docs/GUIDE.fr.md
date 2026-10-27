# hmi-translate — guide d'utilisation

*English version: [GUIDE.md](GUIDE.md).*

1. [L'aller-retour, outil par outil](#1-laller-retour-outil-par-outil)
2. [Moteurs de traduction](#2-moteurs-de-traduction)
3. [Longueur des champs](#3-longueur-des-champs)
4. [Glossaire](#4-glossaire)
5. [Mémoire de traduction](#5-mémoire-de-traduction)
6. [Relecture : ce que signifie chaque contrôle](#6-relecture--ce-que-signifie-chaque-contrôle)
7. [Ligne de commande](#7-ligne-de-commande)
8. [Problèmes courants](#8-problèmes-courants)
9. [Questions fréquentes](#9-questions-fréquentes)

L'interface compte cinq étapes : **Fichiers**, **Langues et moteur**, **Traduire**, **Relire**, **Exporter**. Vous pouvez revenir à n'importe quelle étape ; le projet est enregistré automatiquement et rouvert au démarrage suivant.

## 1. L'aller-retour, outil par outil

Règle générale : **exportez depuis votre outil, traduisez ici, importez d'abord le résultat dans une copie du projet**, puis ouvrez quelques écrans et alarmes dans chaque langue. Le nom des menus peut varier légèrement d'une version à l'autre de chaque outil.

| Outil | Ce qu'il faut exporter | Ce que produit hmi-translate | Ce qu'il faut importer |
|---|---|---|---|
| TIA Portal (STEP 7, WinCC Comfort/Advanced/Unified) — textes du projet | Outils › Exporter les textes du projet (aussi dans Langues et ressources › Textes du projet), langue source et langues cibles cochées, en .xlsx | `<nom>_translated.xlsx` : le même classeur, colonnes cibles remplies (les colonnes manquantes sont ajoutées dans le même style) | Outils › Importer les textes du projet, avec ce fichier |
| TIA Portal — alarmes IHM | Éditeur d'alarmes IHM › Exporter, en .xlsx | `<nom>_translated.xlsx` | Éditeur d'alarmes IHM › Importer |
| FactoryTalk View ME / SE | Tools › Languages › Export, « Unicode text file », un fichier par langue | `<application>_fr_FR.txt` (un fichier par langue cible, même encodage et même mise en page) | Tools › Languages : ajouter la langue à l'application, puis Import avec ce fichier |
| TwinCAT HMI | Les fichiers du dossier `Localization` du projet IHM (`en.localization`, …) | `fr.localization`, `de.localization`… (même ordre des clés, même indentation) | Copier les fichiers dans ce même dossier `Localization` et les ajouter au projet |
| TwinCAT 3 PLC / CODESYS | Projet › Localisation › Créer un modèle de localisation (.pot) | `<nom>.fr_FR.po`, … | Projet › Localisation › Gérer la localisation : ajouter les fichiers .po |
| Ignition (Translation Manager) | Designer › Tools › Translation Manager › Export, format XML (UTF-8) ou PROPERTIES | `<nom>_fr.xml` (ou `.properties`) par langue cible | Translation Manager › Import |
| Tout autre outil (AVEVA, B&R, listes de textes WinCC Unified, outils maison…) | Un tableau .xlsx ou .csv avec une colonne par langue, titrée par un code comme `en-US`, `de-DE`, `fr-FR` | `<nom>_translated.xlsx` / `.csv` | L'import de votre outil |

À savoir :

* **Plusieurs fichiers à la fois** : importez ensemble tous les exports d'une machine (par exemple textes du projet + alarmes IHM + le .pot de l'automate). Un texte présent dans plusieurs fichiers est traduit une seule fois, de la même façon partout.
* **Les traductions déjà présentes** dans les fichiers sont conservées et contrôlées ; cochez « Remplacer les traductions déjà présentes dans les fichiers » à l'étape 2 pour les retraduire.
* **La langue source** est détectée dans les fichiers (titres de colonnes, noms de fichiers, champs de langue) ; modifiez-la à l'étape 2 si besoin.
* **Le contexte** de chaque texte (classe d'alarme, variable de déclenchement, clé, commentaire) est envoyé avec lui au moteur et affiché à la relecture.
* **Codes de langue** : hmi-translate écrit chaque langue dans le style déjà utilisé par le fichier (`fr-FR`, `fr_FR` ou `fr`).

## 2. Moteurs de traduction

Choisissez le moteur à l'étape 2, puis cliquez sur **Tester la connexion** : l'outil fait un vrai appel au moteur (liste des modèles pour Ollama et les serveurs compatibles OpenAI, le modèle pour Anthropic, la consommation de votre clé pour DeepL). Une adresse, une clé ou un nom de modèle faux apparaît donc avant la traduction.

Les textes partent par lots. Si un lot échoue (pas de réponse, réponse refusée par le moteur, erreur du serveur), ses textes sont marqués **Pas de réponse du moteur** avec la raison, et la traduction continue avec le lot suivant ; relancez plus tard pour les compléter. Une clé, une adresse ou un modèle faux, un moteur injoignable dès le départ, ou trois lots en échec d'affilée arrêtent la traduction avec un message clair.

### Ollama (local)

Rien ne quitte votre ordinateur. Installez [Ollama](https://ollama.com), puis téléchargez un modèle une fois :

| Modèle | Téléchargement | Confortable sur | Résultat |
|---|---|---|---|
| `qwen2.5:14b` (par défaut) | environ 9 Go | carte graphique de 12 Go ou plus | bon pour les langues européennes et le chinois |
| `qwen2.5:7b` | environ 5 Go | carte graphique de 8 Go, ou processeur | utilisable, davantage de textes à relire |
| `qwen2.5:32b` | environ 20 Go | carte graphique de 24 Go | le meilleur des trois, plus lent |

```
ollama pull qwen2.5:14b
```

Tout autre modèle de conversation installé dans Ollama peut être saisi dans le champ **Modèle**. Si Ollama tourne sur un autre ordinateur du réseau, indiquez son adresse dans **Adresse** (par exemple `http://192.168.1.20:11434`) et lancez-y Ollama avec `OLLAMA_HOST=0.0.0.0`.

La vitesse dépend surtout de la carte graphique : une carte qui contient tout le modèle va bien plus vite que le processeur. Essayez d'abord le projet d'exemple ou un petit export pour voir la vitesse sur votre poste. La traduction tourne en arrière-plan, affiche sa progression et peut être arrêtée ; ce qui est fait est conservé.

### API compatibles OpenAI

OpenAI, Azure OpenAI, Mistral, LM Studio, vLLM et tout serveur qui propose `/chat/completions`. Renseignez **Adresse** (par exemple `https://api.mistral.ai/v1` ou `http://localhost:1234/v1`), **Modèle** (un modèle proposé par votre compte) et la **clé d'API**.

### API Anthropic Claude, API DeepL

Renseignez la clé d'API. Pour DeepL, les clés gratuites (terminées par `:fx`) utilisent automatiquement l'adresse de l'offre gratuite.

DeepL traduit les textes avec leurs variables protégées, mais ne suit pas de consignes : le glossaire et les longueurs de champ ne lui sont pas transmis. Les contrôles s'appliquent quand même après la traduction : termes et longueurs à reprendre sont signalés à la relecture, comme avec les autres moteurs.

### Clés

Une clé peut être lue dans les variables d'environnement `OPENAI_API_KEY`, `ANTHROPIC_API_KEY` ou `DEEPL_API_KEY` (l'interface affiche « Clé trouvée dans la variable d'environnement »), ou saisie dans l'interface. Une clé saisie reste en mémoire seulement, sauf si vous cochez « Mémoriser la clé sur cet ordinateur » : elle est alors enregistrée dans `settings.json` du dossier de travail (lisible par votre seul utilisateur sous Linux/macOS). Les clés ne sont jamais écrites dans les projets, rapports ou journaux.

Avec un moteur en ligne, les textes et leur contexte sont envoyés au fournisseur, sous votre propre contrat avec lui. Utilisez Ollama pour les projets confidentiels.

### Mémoire seule

N'utilise que les traductions de votre mémoire : utile pour une nouvelle version d'une machine déjà traduite, ou pour voir ce que la mémoire couvre.

## 3. Longueur des champs

Pour chaque texte, la limite vient, dans cet ordre :

1. d'une colonne **Max length** de l'export (les alarmes IHM de TIA Portal et beaucoup de tableaux en ont une : `Max length`, `Maximum length`, `MaxLength`, `Longueur max`, `Max. Länge`…) ;
2. sinon de la règle choisie à l'étape 2 : **longueur source × facteur** (1,3 par défaut), **pas plus long que la source**, **nombre fixe de caractères** ou **pas de limite**.

Les longueurs sont comptées comme l'opérateur les voit : une variable compte pour 4 caractères, un caractère chinois, japonais ou coréen pour 2, et pour un texte sur plusieurs lignes, c'est la plus longue ligne qui compte. Quand une traduction est trop longue, le moteur est sollicité une fois de plus pour une version plus courte (abréviations d'IHM usuelles permises) ; si elle reste trop longue, le texte est signalé à la relecture.

## 4. Glossaire

Le glossaire intégré compte 300 termes (alarmes, états, modes de marche, équipements de procédé, sécurité, opérations) en anglais, français, allemand, espagnol, italien, portugais, polonais et chinois. Téléchargez-le à l'étape 2 (**Télécharger le glossaire intégré**) pour le consulter.

Le glossaire de votre entreprise est un fichier CSV (virgule ou point-virgule, UTF-8) avec une colonne par langue, titrée par le code de langue. Colonnes facultatives : `id`, `domain`, `note`.

```
en,fr,de,note
Reject conveyor,Convoyeur de rejet,Ausschleusband,ligne FL-200
Fault reset,Acquit défaut,Störungsquittierung,
```

Importez-le à l'étape 2. Il est prioritaire sur le glossaire intégré pour un même terme. Les termes de plusieurs mots et tous les termes de votre glossaire sont obligatoires : une traduction qui ne les emploie pas est signalée. Les mots isolés du glossaire intégré sont des suggestions transmises au moteur.

## 5. Mémoire de traduction

Chaque traduction exportée entre dans la mémoire, avec son origine : **humaine** (modifiée ou validée par vous) ou **machine**. Une traduction humaine n'est jamais remplacée par une traduction machine. La mémoire sert en premier à la traduction suivante : le moteur n'est appelé que pour les textes qu'elle ne connaît pas.

Exportez-la en TMX (étape 5) pour la transmettre à une agence de traduction ou à un autre outil de TAO. Le fichier de mémoire est `memory.json` dans le dossier de travail : copiez-le pour donner votre mémoire à un collègue.

## 6. Relecture : ce que signifie chaque contrôle

| Contrôle | Niveau | Signification |
|---|---|---|
| Variable manquante / Variable en trop | erreur | Une variable, une référence de champ ou une balise de la source manque dans la traduction, ou une a été ajoutée. L'import casserait l'alarme. |
| Trop long pour le champ | erreur (limite de l'export) ou avertissement (votre règle) | La plus longue ligne dépasse la limite. |
| Nombre de lignes différent | avertissement | La source a des retours à la ligne que la traduction n'a pas, ou l'inverse. |
| Repère ou identifiant manquant | erreur | Un repère d'équipement comme `P-101` ou `LT101` manque. |
| Nombre manquant | erreur | Un nombre de la source (consigne, seuil, adresse) manque dans la traduction ; seul le séparateur décimal peut changer. |
| Terme du glossaire non utilisé | avertissement | Un terme obligatoire du glossaire n'est pas traduit comme le glossaire l'indique. |
| Identique à la source | avertissement | Un texte de plusieurs mots est revenu non traduit. |
| Deux-points final différent | avertissement | La source se termine par deux-points et pas la traduction, ou l'inverse. |
| Texte de sécurité : relecture humaine obligatoire | relecture | Arrêt d'urgence, danger, avertissement, protecteur, surface chaude, écrasement… Doit être relu par une personne qui maîtrise la langue. Les mots sont reconnus dans le texte source en anglais, français, allemand, espagnol, italien, portugais, néerlandais, suédois, polonais, tchèque, turc, russe, chinois, japonais et coréen. |
| Traduit autrement ailleurs | avertissement | Le même texte source a des traductions différentes dans le projet. |

Les filtres affichent **tous** les textes, ceux **à vérifier**, les **erreurs** seules, les textes de **sécurité**, les textes **trop longs**, les textes **non validés**. Modifiez une traduction sur place : les contrôles sont refaits quand vous quittez le champ. **Valider** marque une traduction comme relue par une personne ; **Valider les textes affichés (sauf erreurs et textes de sécurité)** valide le reste de la page affichée (jusqu'à 100 lignes du filtre courant) en un clic ; les textes de sécurité se valident un par un, après lecture.

L'export reste possible avec des points ouverts : l'étape 5 indique combien de textes comportent encore une erreur, et le rapport les liste, rien n'est caché.

## 7. Ligne de commande

```
hmi-translate [serve] [--port 4870] [--no-open]
hmi-translate translate <fichiers…> --to fr-FR,de-DE [--from en-US] [--engine ollama|openai|anthropic|deepl|memory]
                        [--model …] [--base-url …] [--length ratio:1.3|source|fixed:40|none]
                        [--overwrite] [--out translated] [--fail-on error|review|warning]
hmi-translate check <fichiers…> --to fr-FR [--from en-US] [--out dossier] [--fail-on …]
hmi-translate glossary --export <fichier.csv> | --import <fichier.csv> | --clear
hmi-translate memory --export <fichier.tmx>
```

`translate` écrit les fichiers traduits, `translation-report.html` et `translation-issues.csv` dans le dossier de sortie. `check` contrôle les traductions déjà présentes dans les fichiers (utile dans une chaîne de build, ou au retour d'une agence de traduction). Codes de sortie : 0 terminé, 1 erreur (fichier, moteur), 2 il reste des points du niveau `--fail-on`.

## 8. Problèmes courants

| Symptôme | Solution |
|---|---|
| « Ollama is not reachable » | Lancez Ollama (icône dans la zone de notification sous Windows, ou `ollama serve`). Vérifiez l'adresse à l'étape 2. |
| « Model … is not installed in Ollama » | Lancez la commande `ollama pull` indiquée dans le message, ou choisissez un des modèles installés qu'il liste. |
| 401 / 403 avec un moteur en ligne | La clé d'API est fausse ou n'a pas accès à ce modèle. |
| 404 avec un moteur en ligne | L'adresse ou le nom du modèle est faux : utilisez un modèle proposé par votre compte. |
| 429 | Le fournisseur limite votre débit : hmi-translate fait jusqu'à 3 tentatives par lot, en attendant un peu plus à chaque fois ; les textes restés sans réponse sont marqués, relancez plus tard pour eux (ce qui est fait est conservé). |
| « Pas de réponse du moteur » sur quelques textes | La raison est indiquée sur chaque texte (erreur du serveur, refus). Cliquez de nouveau sur **Traduire** : seuls les textes manquants partent. |
| « Format not recognised » | Vérifiez l'extension, et que le fichier vient bien de la fonction d'export de votre outil (pas d'une copie collée dans un autre classeur). |
| Fichier .xls refusé | Enregistrez-le en .xlsx dans Excel, ou refaites l'export en choisissant .xlsx. |
| Accents faux après import dans FactoryTalk View | Utilisez l'export « Unicode text file » ; hmi-translate garde l'encodage du fichier source. |
| Le navigateur ne s'ouvre pas | Ouvrez `http://127.0.0.1:4870` à la main. Si le port est pris, hmi-translate prend le suivant libre et l'affiche dans la fenêtre. |
| L'essai s'arrête à 500 textes | L'essai traduit 500 textes distincts par projet ; les autres sont signalés « Au-delà des 500 textes de l'essai ». |

## 9. Questions fréquentes

**Mon projet est-il envoyé quelque part ?** Pas avec Ollama ni avec la mémoire seule : tout reste sur votre ordinateur. L'interface n'écoute que sur `127.0.0.1` et refuse les requêtes venant d'autres sites. Avec un moteur en ligne, les textes ne sont envoyés qu'à ce fournisseur.

**Change-t-il autre chose dans mes fichiers ?** Non. Dans les .xlsx, seules les cellules traduites sont réécrites ; mise en forme, autres feuilles et colonnes restent telles quelles. Les formats texte gardent leur encodage, leurs fins de ligne, leur ordre et leurs commentaires.

**Quelles langues ?** Toute langue connue du moteur. L'interface en propose 37 courantes et accepte tout code (`ro-RO`, `sv-SE`, `ar-SA`…). Le glossaire intégré en couvre 8.

**Une agence de traduction peut-elle relire le résultat ?** Oui : donnez-lui `translation-issues.csv` ou le .xlsx traduit, puis lancez `hmi-translate check` sur ce qu'elle renvoie. L'export TMX transporte votre mémoire vers ses outils.

**Plusieurs ingénieurs ?** Chacun installe hmi-translate sur son poste. Partagez le CSV du glossaire d'entreprise et le fichier de mémoire, ou faites pointer `HMI_TRANSLATE_HOME` vers un dossier partagé utilisé par une personne à la fois.

**Est-il certifié pour les textes de sécurité ?** Non. Il les signale pour relecture, et ils ne peuvent être validés qu'un par un, par une personne.

**Comment la licence est-elle vérifiée ?** Elle ne l'est pas : aucune activation, rien ne nous est envoyé. L'édition complète se télécharge après l'achat et s'utilise selon [LICENSE.md](../LICENSE.md) (une personne nommée, ou jusqu'à 5 pour la licence Constructeur, pendant 12 mois).
