# Tests de recrutement IT — CCG Flow

**Un seul lien : https://siradio.github.io/ccg-tests-it/**

Une seule page, un seul formulaire de connexion. **C'est l'identifiant qui
décide de ce qui s'ouvre.**

Chaque candidat a son identifiant, qui lui ouvre sa variante et elle seule.
Le responsable IT et les RH en ont un autre, qui ouvre la page des résultats.
Le mot de passe est commun.

**Ni les identifiants ni le mot de passe ne figurent dans ce dépôt** — pas
davantage que les noms des candidats. Tout cela se transmet séparément.

## Comment ça tient

La page contient cinq blocs chiffrés en AES-256-GCM, avec un sel commun. À la
saisie, une seule clé est dérivée du couple identifiant + mot de passe par
PBKDF2-HMAC-SHA256 (310 000 itérations), puis essayée sur chaque bloc : celui
qui s'authentifie est le bon.

Rien dans la page ne dit quel identifiant correspond à quel bloc — leur ordre
est même brassé à la génération. Un candidat ne peut ni deviner l'existence de
la page de résultats, ni ouvrir le test d'un autre.

Le fichier publié ne contient aucun texte lisible : ni les questions, ni les
noms des candidats, ni le corrigé.

## Le corrigé n'est pas dans les tests

C'est le point de conception le plus important.

Dans le fichier d'origine, la bonne réponse de chaque question était écrite
dans le code de la page : un candidat qui ouvrait le code source lisait le
corrigé. Les onze dernières questions avaient même toutes leur bonne réponse
en première position.

Les blocs des candidats ne contiennent **que les énoncés et les options**, dans
un ordre brassé à la génération puis rebrassé à chaque passage. La page
enregistre le **libellé** choisi, jamais une position ni une appréciation.

Le corrigé vit uniquement dans le bloc `rh`. La notation se fait après coup, en
comparant les libellés.

## Le parcours

1. Le candidat ouvre le lien, saisit l'identifiant qu'on lui a remis et le
   mot de passe.
2. Il complète ses coordonnées et démarre. Le chronomètre de 40 minutes part et
   ne se met pas en pause.
3. Il répond aux 30 questions, valide le QCM — confirmation demandée, avec le
   nombre de questions sans réponse.
4. Il répond aux 3 questions ouvertes, soumet — **seconde confirmation**.
5. Les réponses partent par courriel au responsable IT, les RH en copie.
6. Le responsable ouvre le même lien avec `rh` et retrouve la copie.

Si le chronomètre arrive à zéro, le QCM est validé automatiquement avec les
réponses déjà saisies.

## La page des résultats

Elle liste les copies : candidat, variante, date, durée, total sur 30 et note
sur 20. Un clic sur une ligne ouvre le détail — scores par domaine, les
30 réponses avec l'attendu en regard des erreurs, et les questions ouvertes.

Les copies sont conservées dans le navigateur, si bien que la page se remplit
au fil des passages.

### Relève automatique

Pour éviter d'avoir à coller chaque copie à la main, la page sait les relever
toute seule — il lui faut une adresse où lire. GitHub Pages est un hébergement
**statique** : il n'a ni base, ni stockage.

Le plus simple est un script Google Apps Script relié à une feuille de calcul,
gratuit et déployé en quelques minutes :

```js
// Feuille de calcul > Extensions > Apps Script. Déployer en application web,
// exécutée par vous, accessible à « tout le monde ».
function doPost(e) {
  const f = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
  f.appendRow([new Date(), e.postData.contents]);
  return ContentService.createTextOutput('ok');
}
function doGet() {
  const f = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
  const copies = f.getDataRange().getValues().slice(1)
    .map(l => { try { return JSON.parse(l[1]); } catch (x) { return null; } })
    .filter(Boolean);
  return ContentService.createTextOutput(JSON.stringify(copies))
    .setMimeType(ContentService.MimeType.JSON);
}
```

L'adresse de déploiement se renseigne ensuite en tête du bloc `rh`, avant
chiffrement :

```js
var COLLECTE = { url: "https://script.google.com/macros/s/..../exec" };
```

Tant qu'elle est vide, la page fonctionne par ajout manuel.

## L'envoi du courriel

Il passe par [FormSubmit](https://formsubmit.co), qui ne demande aucun compte.

**Au tout premier envoi**, FormSubmit adresse un courriel d'activation au
premier destinataire. Tant que ce lien n'est pas confirmé, rien n'arrive.
Faites un envoi de test avant de convoquer les candidats.

Les destinataires se changent en tête de chaque bloc candidat :

```js
var CONFIG = {
  destinataires: ["it@ccggroupe.com", "rh@ccggroupe.com"],
  service: "formsubmit"
};
```

Si l'envoi échoue, le candidat n'est pas bloqué : un bouton ouvre son logiciel
de messagerie, pré-adressé, et le `.json` reste téléchargeable.

## Limites à connaître

- Ce dépôt est public : le fichier chiffré est téléchargeable par tous. La
  confidentialité repose sur la robustesse du mot de passe, une attaque hors
  ligne restant toujours possible.
- Le mot de passe étant commun, **seul l'identifiant sépare un candidat de la
  page des résultats**.
- Le chronomètre est côté navigateur. Recharger la page le remet à zéro. Ces
  tests supposent une surveillance.
- Aucune reprise en cas de fermeture d'onglet.

## Régénérer

Les générateurs et les sources non chiffrées sont conservés hors de ce dépôt.

```bash
python extract_tests.py    # lit les quatre fichiers d'origine
python mktest.py           # blocs candidat, sans corrigé
python mkresultats.py      # bloc résultats, avec le corrigé
node mkportail.js index.html "Tests de recrutement IT" \
     tests-pub/test-a.html:<identifiant>:<motdepasse> \
     tests-pub/test-b.html:<identifiant>:<motdepasse> \
     tests-pub/test-c1.html:<identifiant>:<motdepasse> \
     tests-pub/test-c2.html:<identifiant>:<motdepasse> \
     tests-pub/resultats.html:<identifiant>:<motdepasse>
```
