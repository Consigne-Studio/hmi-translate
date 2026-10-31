# hmi-translate

Traduction par IA des textes d'IHM et d'automate (alarmes, écrans, boutons, commentaires) qui préserve les variables d'alarme, la longueur des champs, les repères et votre glossaire, et rend des fichiers que votre atelier logiciel réimporte tels quels.

*English version: [README.md](README.md).*

**Téléchargez l'essai gratuit** dans [Releases](../../releases) : toutes les fonctions, 500 textes par projet. Paquet Windows avec Node.js inclus (décompresser, double-cliquer sur `Start HMI Translate.cmd`), ou paquet portable Linux / macOS (Node.js 20 ou plus).

**Licences** : Ingénieur (une personne) ou Constructeur (5 personnes), 12 mois : [consignestudio.gumroad.com/l/hmi-translate](https://consignestudio.gumroad.com/l/hmi-translate).

Formats : TIA Portal / WinCC (.xlsx), FactoryTalk View ME/SE (.txt Unicode), TwinCAT HMI (.localization), TwinCAT 3 PLC / CODESYS (.pot/.po), Ignition (.xml/.properties), tout tableau .xlsx/.csv avec une colonne par langue.

Moteurs : Ollama sur votre ordinateur (rien n'en sort), API compatibles OpenAI, API Anthropic Claude, API DeepL avec votre clé, ou mémoire de traduction seule.

Documentation : [guide d'utilisation](docs/GUIDE.fr.md), [changements](CHANGELOG.md).

Limites : formats réalisés d'après la documentation des éditeurs et la structure de leurs exports, testés sur des fichiers construits selon cette structure. Faites le premier import dans une copie du projet. Les textes de sécurité doivent être relus par une personne qui maîtrise la langue avant la mise en service.

Ce dépôt contient la documentation et les téléchargements de l'essai ; hmi-translate n'est pas un logiciel libre ([LICENSE](LICENSE)).

Support : support@consignestudio.fr
