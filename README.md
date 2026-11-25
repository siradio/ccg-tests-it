# Tests de recrutement IT — CCG Flow

Quatre tests de recrutement, un par candidat, servis par GitHub Pages :
**https://siradio.github.io/ccg-tests-it/**

| Page | Qui | Contenu |
|---|---|---|
| `test-a.html` … `test-c2.html` | un candidat chacune | 30 QCM chronométrés + 3 questions ouvertes |
| `correction.html` | responsable IT et RH | le corrigé des quatre variantes, et la notation |
| `index.html` | public | une page muette qui renvoie vers le lien personnel |

## Accès protégé

Chaque page est **chiffrée** : AES-256-GCM, clé dérivée de l'identifiant et du
mot de passe par PBKDF2-HMAC-SHA256 à 310 000 itérations, déchiffrement dans
le navigateur. Le fichier publié ne contient aucun texte lisible — ni les
questions, ni les noms des candidats.

Chaque candidat a **son mot de passe**. La page de correction a le sien, qui
n'est pas celui des tests.

**Les identifiants ne figurent pas dans ce dépôt.** Ils sont transmis
séparément.

## Le corrigé ne se trouve pas dans les pages des candidats

C'est le point de conception le plus important.

Dans le fichier d'origine, la bonne réponse de chaque question était écrite
dans le code de la page : un candidat qui ouvrait le code source lisait le
corrigé. Pire, les onze dernières questions avaient toutes leur bonne réponse
en première position.

Les pages publiées ici ne contiennent **que les énoncés et les options**, dans
un ordre brassé à la génération puis rebrassé à chaque passage. La page
enregistre le **libellé** choisi par le candidat, jamais une position ni une
appréciation.

Le corrigé vit uniquement dans `correction.html`. La notation se fait après
coup, en comparant les libellés.

## Le parcours

1. Le candidat ouvre son lien, saisit identifiant et mot de passe.
2. Il complète ses coordonnées, lit les consignes, démarre. Le chronomètre de
   40 minutes part et ne se met pas en pause.
3. Il répond aux 30 questions, valide le QCM — une confirmation est demandée,
   avec le nombre de questions sans réponse.
4. Il répond aux 3 questions ouvertes, soumet — **seconde confirmation**.
5. Les réponses partent par courriel au responsable IT, avec les RH en copie.
6. Le responsable ouvre `correction.html`, y colle le champ `JSON` du courriel
   reçu, et obtient la note, le détail par domaine et les réponses fausses.

Si le chronomètre arrive à zéro, le QCM est validé automatiquement avec les
réponses déjà saisies, et les questions ouvertes restent accessibles.

## L'envoi du courriel

GitHub Pages est un hébergement **statique** : il n'y a pas de serveur, donc
pas d'envoi de courriel possible par la page seule. L'envoi passe par
[FormSubmit](https://formsubmit.co), qui ne demande aucun compte.

**Au tout premier envoi**, FormSubmit adresse un courriel d'activation au
premier destinataire. Tant que ce lien n'est pas confirmé, rien n'arrive.
Faites un envoi de test avant de convoquer les candidats.

Les destinataires se changent en tête de chaque page, avant chiffrement :

```js
var CONFIG = {
  destinataires: ["it@ccggroupe.com", "rh@ccggroupe.com"],
  service: "formsubmit"
};
```

Le premier reçoit, les suivants sont en copie.

### Si l'envoi échoue

Le candidat n'est pas bloqué. La page affiche un bouton qui ouvre son
logiciel de messagerie, pré-adressé aux deux destinataires, et un
téléchargement `.json` qu'il peut transmettre autrement. Le `.json` est aussi
ce que la page de correction accepte directement.

## Limites à connaître

- Ce dépôt est public : les fichiers chiffrés sont téléchargeables par tous.
  La confidentialité repose sur la robustesse des mots de passe, une attaque
  hors ligne restant toujours possible.
- Le chronomètre est côté navigateur. Il n'empêche ni de recharger la page
  pour repartir à zéro, ni de chercher les réponses ailleurs. **Ces tests
  supposent une surveillance**, ou une tolérance assumée sur ce point.
- Aucune reprise en cas de fermeture d'onglet : le test recommence.

## Régénérer

Les générateurs et les sources non chiffrées sont conservés hors de ce dépôt.

```bash
python extract_tests.py      # lit les quatre fichiers d'origine
python mktest.py             # pages candidat, sans corrigé
python mkcorr.py             # page de correction et accueil
node mkgate.js <source> <sortie> <identifiant> <motdepasse> "<titre>"
```
