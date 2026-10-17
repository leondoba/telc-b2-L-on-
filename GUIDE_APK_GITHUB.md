# Guide complet — site hors ligne, installable, et APK Android via GitHub

Contenu du dossier :

```
telc-b2-repo/
├─ www/                       ← le site (index.html + manifest + service worker + icônes)
├─ package.json               ← dépendances Capacitor (emballage Android)
├─ capacitor.config.json      ← nom et identifiant de l'application
└─ .github/workflows/
   ├─ build-apk.yml           ← construit l'APK dans le cloud GitHub
   └─ pages.yml               ← publie le site en ligne (GitHub Pages)
```

> **Important (droits d'auteur).** Le site contient des exercices issus de PDF de tiers (académies, auteurs de recueils). Pour un usage **personnel ou en classe privée**, pas de souci pratique. Si vous publiez le site ou l'APK **publiquement**, retirez ou faites valider ces contenus, ou laissez le dépôt **privé**.

---

## 1. Ouvrir le site hors ligne, avec n'importe quel navigateur (le plus simple)

`www/index.html` est **un fichier unique autonome** : toutes les données (exercices, corrigés) sont dedans, plus aucune police ni script externe.

1. Copiez `index.html` sur le téléphone ou l'ordinateur (clé USB, WhatsApp, Drive…).
2. Ouvrez-le avec Chrome, Edge, Firefox, Safari, Brave, Samsung Internet… (double-clic ou « Ouvrir avec »).
3. Tout fonctionne sans internet : exercices, corrections, chronomètres, examens blancs, progression (enregistrée dans le navigateur).

Limites en ouverture directe (`file://`) :
- la progression est liée à ce fichier et à ce navigateur (autre navigateur = progression vide) ;
- l'installation « comme une app » n'est pas proposée (il faut une adresse web, voir §3) ;
- les vidéos YouTube exigent internet ; pour du vrai hors ligne, chargez un **fichier audio** (bouton « fichier » de la zone audio) : il est mémorisé dans le navigateur ;
- la voix de synthèse dépend des voix allemandes installées sur l'appareil (Android : Paramètres › Synthèse vocale › ajouter l'allemand).

---

## 2. Créer le dépôt GitHub

1. Créez un compte sur https://github.com (gratuit).
2. Bouton **New repository** → nom `telc-b2-simulator` → **Private** (recommandé) → **Create repository**.
3. Envoyez les fichiers :
   - *Sans ligne de commande* : sur la page du dépôt, **Add file › Upload files**, glissez **tout le contenu** du dossier `telc-b2-repo` (y compris le dossier caché `.github` — si le glisser-déposer l'ignore, créez les deux fichiers à la main : **Add file › Create new file**, nom `.github/workflows/build-apk.yml`, collez le contenu), puis **Commit changes**.
   - *Avec Git* :
     ```bash
     cd telc-b2-repo
     git init -b main
     git add .
     git commit -m "Première version"
     git remote add origin https://github.com/VOTRE_NOM/telc-b2-simulator.git
     git push -u origin main
     ```
4. Vérifiez que `www/index.html`, `package.json` et `.github/workflows/` sont bien visibles sur GitHub.

---

## 3. Option A — Site en ligne + « application » installable et hors ligne (sans APK)

C'est la solution la plus robuste : un seul lien, tous les moteurs, mises à jour automatiques.

1. Dépôt › **Settings › Pages › Build and deployment › Source : GitHub Actions**.
2. Onglet **Actions** : le workflow *Publier le site* se lance à chaque envoi sur `main` (ou **Run workflow**).
3. Au bout d'une minute, l'adresse apparaît : `https://VOTRE_NOM.github.io/telc-b2-simulator/`. (Dépôt privé : Pages privé nécessite un plan payant ; sinon dépôt public.)
4. Ouvrez l'adresse **une fois avec internet** : le *service worker* (`sw.js`) met tout en cache. Ensuite le site s'ouvre **sans connexion**, même après redémarrage.
5. Installer :
   - **Android / Chrome, Edge, Samsung Internet** : menu ⋮ › **Installer l'application** (ou le bouton « 📲 Installer l'application » en bas de page).
   - **iPhone / iPad (Safari)** : Partager › **Sur l'écran d'accueil** (c'est la seule voie sur iOS : il n'existe pas d'APK pour iPhone).
   - **Ordinateur (Chrome/Edge)** : icône d'installation dans la barre d'adresse.

---

## 4. Option B — Fabriquer l'APK Android avec GitHub Actions

Le fichier `.github/workflows/build-apk.yml` utilise **Capacitor** (emballe le site dans une application Android) et compile dans le cloud GitHub : vous n'installez ni Android Studio ni Java.

### 4.1 Lancer la compilation
1. Dépôt › onglet **Actions**.
2. À gauche : **Build Android APK** › bouton **Run workflow** › **Run workflow** (branche `main`).
3. Attendez 5 à 10 minutes (pastille jaune → verte ✔). Rouge ✘ : ouvrez l'exécution pour lire l'erreur (voir §6).
4. Dans l'exécution terminée, section **Artifacts** : téléchargez **telc-b2-apk** (un `.zip` contenant `app-debug.apk`).

### 4.2 Installer sur le téléphone
1. Dézippez, envoyez `app-debug.apk` sur le téléphone.
2. Ouvrez-le ; si Android demande une autorisation : **Autoriser l'installation depuis cette source** (Paramètres › Applications › Accès spécial › Installer des applis inconnues).
3. Si **Play Protect** avertit : « Installer quand même » (l'APK est signé avec une clé de test, c'est normal).
4. L'application **« telc B2 »** apparaît avec son icône. Elle fonctionne **hors ligne** : le site est embarqué dans l'APK.

### 4.3 Publier une version numérotée
```bash
git tag v1.0.0
git push --tags
```
Chaque tag `v…` relance la compilation.

### 4.4 Mettre à jour le contenu
1. Remplacez `www/index.html` (nouvelle version du site).
2. Dans `www/sw.js`, changez `const VERSION = 'telcb2-v3'` en `'telcb2-v4'` (force le rafraîchissement du cache).
3. Commit + push → le site Pages se met à jour tout seul ; relancez **Build Android APK** pour un nouvel APK.

---

## 5. Pour aller plus loin

### APK signé « release » (partage large) et Google Play
- Le `app-debug.apk` suffit pour un usage personnel et la classe.
- Pour Google Play il faut un **AAB signé** et un compte développeur (25 USD, paiement unique). Créez une clé : `keytool -genkeypair -v -keystore telc.jks -alias telc -keyalg RSA -keysize 2048 -validity 10000`, **conservez-la** (sans elle, plus de mise à jour), stockez-la en *Secrets* GitHub (`KEYSTORE_BASE64`, mots de passe) et remplacez `assembleDebug` par `bundleRelease` + une étape de signature (par ex. l'action `r0adkll/sign-android-release`). N'ajoutez jamais la clé au dépôt (`.gitignore` l'exclut).

### Alternative sans Capacitor : TWA (PWABuilder)
Si le site est en ligne (§3), https://www.pwabuilder.com › entrez l'adresse › **Package for stores › Android** : télécharge un projet/APK basé sur le site en ligne (il faut internet au premier lancement pour le chargement, puis le cache prend le relais).

### Compatibilité des moteurs
- **Chromium** (Chrome, Edge, Brave, Opera, Samsung Internet), **Gecko** (Firefox), **WebKit** (Safari, iOS) : le code n'utilise que des fonctions standard.
- Testé ici en **Chromium** (navigation, correction, minuteries, hors ligne avec service worker). Firefox et Safari non testés dans cet environnement : si une case se comporte mal chez vous, notez le navigateur + la version.
- Navigateurs de plus de ~2020 : mettre à jour (le site utilise des fonctions JavaScript modernes).

---

## 6. Problèmes fréquents

| Symptôme | Cause / solution |
|---|---|
| Actions : « No such file… android » | Le workflow crée le dossier `android` lui-même (`npx cap add android`). Vérifiez que `package.json` et `capacitor.config.json` sont à la **racine** du dépôt. |
| Étape Gradle en erreur (licence/SDK) | Relancez ; sinon dans l'exécution, copiez les 20 dernières lignes d'erreur et envoyez-les-moi. |
| L'APK s'installe mais écran blanc | Vérifiez que `www/index.html` existe sur GitHub avant la compilation ; relancez le workflow. |
| Pas de son (voix de synthèse) | Installer la voix allemande dans les réglages de synthèse vocale d'Android, puis relancer l'app. |
| YouTube « vidéo indisponible » | Lecture intégrée refusée par la vidéo, ou hors ligne : utilisez le bouton « Ouvrir sur YouTube » ou chargez un **fichier audio**. |
| Progression perdue | Vider les données du navigateur/app efface la progression ; elle n'est pas synchronisée entre appareils. |
| Page Pages « 404 » | Pages › Source = **GitHub Actions** ; attendre la fin du workflow *Publier le site*. |
| Le site ne se met pas à jour sur l'appareil | Changer `VERSION` dans `sw.js`, puis fermer/rouvrir (ou recharger deux fois). |

> Ce que je n'ai pas pu tester ici : la compilation Android elle-même (pas d'accès à npm/Android SDK dans cet environnement). Les fichiers suivent la procédure Capacitor 6 standard ; à la première exécution, si une étape échoue, envoyez-moi le journal d'erreur et je corrige.


---

## Annexe — Version 4 : sécurité et contenu

- **CSP stricte** (balise `<meta http-equiv="Content-Security-Policy">`) : aucun script externe, scripts inline autorisés uniquement par empreinte SHA-256, pas d'`eval`, pas de formulaires, `object-src 'none'`. Seuls les iframes YouTube (domaine sans cookies) sont permis.
- **Important :** si vous modifiez un script dans `index.html`, les empreintes ne correspondent plus et le site cesse de fonctionner. Régénérez-les (script `patch4.py`) ou demandez une nouvelle version.
- **Données :** tout est local (localStorage / IndexedDB), aucun compte, aucun suivi. Boutons « Sauvegarder / Restaurer » sur l'accueil (fichier JSON validé à l'import).
- **Module Monde :** 8 entraînements originaux (non officiels, non affiliés aux organismes).
- Pour publier une nouvelle version : modifiez `VERSION` dans `www/sw.js` (ici `telcb2-v10`).
