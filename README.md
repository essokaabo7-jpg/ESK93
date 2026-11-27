# ESK93 — version publique (Firebase)

Place de marché ouverte à tous : chacun crée son compte avec **e-mail + mot de passe**,
publie ses articles, achète, discute en privé avec le vendeur, confirme la livraison et note le vendeur.
Une **commission de 5 %** est calculée sur chaque vente (à envoyer au +237 653 423 053).

Contenu du dossier :

| Fichier | Rôle |
|---|---|
| `dist/index.html` | Toute l’application (à configurer à l’étape 3) |
| `dist/manifest.webmanifest`, `sw.js`, `icon-*.png`, `icon.svg` | Installation sur l’écran d’accueil du téléphone |
| `firestore.rules` | Règles de sécurité : qui peut lire et écrire quoi |
| `firebase.json` | Réglages d’hébergement Firebase |

Coût : l’offre gratuite de Firebase (Spark) suffit pour démarrer.

---

## Étape 1 — Créer le projet Firebase

1. Va sur https://console.firebase.google.com et connecte-toi avec un compte Google.
2. **Ajouter un projet** → nom : `esk93` → continue (Google Analytics : facultatif, tu peux refuser).

## Étape 2 — Activer les comptes et la base de données

1. **Build → Authentication → Commencer → Adresse e-mail/Mot de passe → Activer** → Enregistrer.
2. **Build → Firestore Database → Créer une base de données** → choisis la région la plus proche
   (ex. `europe-west1`) → **mode production** → Activer.
3. Dans Firestore, onglet **Règles** : efface tout, colle le contenu du fichier `firestore.rules`, clique **Publier**.

## Étape 3 — Récupérer la configuration et la coller dans l’application

1. Roue dentée (⚙) → **Paramètres du projet** → section « Vos applications » → icône **`</>` (Web)**.
2. Nom : `esk93-web` → **Enregistrer l’application**. Firebase affiche un bloc `firebaseConfig = { ... }`.
3. Ouvre `dist/index.html`, cherche `COLLE_ICI` (tout en haut du script) et remplace chaque valeur
   par celles affichées par Firebase.

## Étape 4 — Mettre le site en ligne (Firebase Hosting)

Sur ton ordinateur, avec [Node.js](https://nodejs.org) installé, dans le dossier `esk93-public` :

```bash
npm install -g firebase-tools
firebase login
firebase use --add        # choisis ton projet esk93
firebase deploy
```

Firebase te donne une adresse du type `https://esk93.web.app` : envoie-la à tout le monde,
elle fonctionne dans Chrome, Safari, etc., **sans compte Claude**.
(Plus tard : Hosting → « Ajouter un domaine personnalisé » pour avoir ton propre nom de domaine.)

## Étape 5 — Devenir administrateur (voir les commissions)

1. Ouvre le site, crée ton compte ESK93 et vérifie ton e-mail.
2. Firebase console → **Authentication → Utilisateurs** → copie ton **UID**.
3. **Firestore → Démarrer une collection** → ID : `admins` → ID du document : *ton UID* →
   ajoute un champ `ok` (booléen) = `true` → Enregistrer.
4. Recharge le site : l’onglet **Mon compte → Commissions** apparaît.
   Tu y vois toutes les ventes, le total des commissions et tu peux marquer « Reçue ».

---

## Comment ça marche

- **Inscription** : nom, e-mail, mot de passe (+ téléphone Mobile Money facultatif). Un e-mail de vérification est envoyé.
  Il faut l’avoir validé pour **publier** une annonce (règle anti-spam).
- **Annonces** : visibles par tous, même sans compte. Jusqu’à 3 photos par annonce.
- **Achat** : « Acheter » réserve l’article et ouvre une discussion ; le vendeur confirme la vente avec le prix final.
- **Messages** : privés. Les règles Firestore n’autorisent que l’acheteur et le vendeur à lire une discussion.
- **Livraison et avis** : le vendeur marque « Livré », l’acheteur confirme la réception, puis note le vendeur de 1 à 5.
- **Commission 5 %** : calculée à la confirmation de vente et affichée au vendeur. L’application **ne déplace pas
  d’argent** : le vendeur envoie la commission au +237 653 423 053, puis tu la marques « Reçue ».

## À savoir avant de lancer

- **Paiements** : pour que l’argent de l’achat et la commission passent automatiquement par Orange Money / MTN MoMo,
  il faudra intégrer un service de paiement Mobile Money (via un développeur et un petit serveur). Ce n’est pas inclus.
- **Retirer l’obligation d’e-mail vérifié** : dans `firestore.rules`, supprime la ligne
  `&& request.auth.token.email_verified == true`, puis republie les règles.
- **Photos** : stockées directement dans la base (compressées). Au-delà de quelques milliers d’annonces,
  passe à Firebase Storage (offre payante Blaze).
- **Notifications** : elles s’affichent quand le site est ouvert. Les notifications push téléphone demandent
  Firebase Cloud Messaging (à ajouter plus tard).
- **Modération** : il n’y a pas encore de bouton de signalement ni de suspension d’utilisateur.
- **Légal** : avant d’ouvrir au public, prévois des conditions d’utilisation et une politique de confidentialité
  (tu collectes des e-mails et des numéros de téléphone).
- **Sécurité** : ne partage jamais ton compte Google/Firebase. La clé `apiKey` dans `index.html` n’est pas secrète :
  la protection vient des règles Firestore.


---

## Partage et installation

- **Partager ESK93** : bouton de partage en haut de l’écran et bouton « Inviter des amis » dans Mon compte
  (WhatsApp, Facebook, SMS, copier le lien, ou le menu de partage du téléphone).
- **Partager un article** : bouton « Partager l’article » sur chaque fiche. Le lien (`…/#a=identifiant`) ouvre
  directement l’article, même pour quelqu’un qui n’a pas encore de compte.
- **Installer comme une application** : sur Chrome (Android), menu ⋮ → « Installer l’application » ;
  sur iPhone (Safari), Partager → « Sur l’écran d’accueil ».

## Profil et retrait Mobile Money

- **Photo de profil** : Mon compte → « Changer la photo » (recadrée en carré et compressée).
- **Mon retrait Mobile Money** : chaque utilisateur enregistre son opérateur (MTN Mobile Money ou Orange Money)
  et son numéro. Ces données sont privées.
- **Paiement en ligne : pas encore actif.** Pour envoyer de l’argent dans l’application puis le retirer vers
  MTN ou Orange Money, il faut :
  1. un compte chez un service de paiement Mobile Money agréé (agrégateur) pour le Cameroun ;
  2. un petit serveur (Firebase Cloud Functions) qui garde les clés secrètes et appelle ce service ;
  3. de vérifier le cadre légal : conserver l’argent des utilisateurs dans un « portefeuille » est une activité
     réglementée. La solution la plus simple et sûre est de laisser le service de paiement encaisser l’acheteur
     et reverser le vendeur (en retenant les 5 %), sans que l’application garde un solde.
  Quand tu auras ce compte, le code se branche à l’endroit des boutons « Acheter » et « Confirmer la vente ».

## Beaucoup d’utilisateurs

- La liste charge **60 annonces à la fois** (bouton « Voir plus ») avec de petites vignettes ; les photos
  complètes ne se chargent qu’à l’ouverture d’une fiche.
- Firebase gratuit (Spark) : quotas quotidiens de lecture/écriture (de l’ordre de 50 000 lectures par jour).
  Si l’application décolle, passe au forfait **Blaze** (paiement à l’usage, avec budget d’alerte réglable).
- Pense à ajouter plus tard : signalement et modération, notifications push, recherche côté serveur.

## Mettre à jour un site déjà publié

Après avoir remplacé les fichiers : republie les règles (Firestore → Règles → coller `firestore.rules` → Publier),
puis `firebase deploy`.
