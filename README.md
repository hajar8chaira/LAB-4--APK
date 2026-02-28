# LAB 4 — Analyse statique d’un APK avec JADX GUI (+ dex2jar + JD-GUI)



Deux APK ont été utilisés :
1) `DivaApplication.apk` (application pédagogique vulnérable)
2) `app-debug.apk` (APK généré via Android Studio pour comparaison)

---

## 2. Workspace et vérifications initiales (Task 1)
Dossier de travail utilisé :

- `C:\APK-Analysis`

### 2.1 Vérification du format APK (signature ZIP)
Commande PowerShell  :
- `Get-Content -Path .\DivaApplication.apk -TotalCount 4 | Format-Hex`

Résultat attendu :
- Les premiers octets commencent par `50 4B` (équivalent "PK"), ce qui confirme que l’APK est une archive ZIP valide.

### 2.2 Listing de la structure interne (premières entrées)
Commande PowerShell utilisée :
- `Add-Type -Assembly System.IO.Compression.FileSystem`
- `[System.IO.Compression.ZipFile]::OpenRead($apk.FullName).Entries.FullName | Select-Object -First 20`

Résultats observés (exemples) :
- `AndroidManifest.xml`
- `res/...` (fichiers ressources)
- présence de `classes.dex` et dex multiples pour certains APK (ex : `classes2.dex`, `classes3.dex`, `classes4.dex`)

Conclusion :
- Les APK sont lisibles, structurés correctement, et contiennent les éléments attendus (manifest, res, dex).

### 2.3 Hash SHA-256 (traçabilité)
Commande PowerShell :
- `Get-FileHash -Algorithm SHA256 <apk>`

But :
- conserver un identifiant de version du fichier analysé (traçabilité, comparaison, reproductibilité).

---

## 3. Vérification de signature (Task 1 — optionnel)
### 3.1 DIVA (`DivaApplication.apk`)
Vérification via `apksigner` (Android SDK Build-Tools) :
- `apksigner verify --verbose "C:\APK-Analysis\DivaApplication.apk"`

Résultat observé (résumé) :
- Signature v1 : true
- v2/v3/v4 : false
- Number of signers : 1

Interprétation :
- DIVA est une application ancienne : présence d’une signature v1 uniquement (JAR signing).

### 3.2 APK Android Studio (`app-debug.apk`)
Vérification via `apksigner` :
- `apksigner verify --verbose "C:\APK-Analysis\app-debug.apk"`

Résultat observé (résumé) :
- Signature v1 : false
- Signature v2 : true
- v3/v3.1/v4 : false
- Number of signers : 1

Interprétation :
- APK moderne : signature v2 active (schéma plus récent et recommandé), v1 non présent (cas possible selon config du build).

·· APK: app-debug.apk 
## 1. Informations Générales

- Nom du fichier : app-debug.apk
- Localisation : C:\APK-Analysis
- Type : APK (archive ZIP valide)
- Signature ZIP : OK (50 4B — PK)
- SHA-256 :
  4F51CD7C10CA869A465513150F38272E006FB77BC898F16F987E0105AE4404FE

---

## 2. Vérification de la Signature Android

Commande utilisée :
apksigner verify --verbose app-debug.apk

Résultats :

- Verified using v1 scheme (JAR signing) : false
- Verified using v2 scheme (APK Signature Scheme v2) : true
- Verified using v3 scheme : false
- Verified using v4 scheme : false
- Number of signers : 1

### Analyse

L'application est signée avec le schéma v2 uniquement.

Le schéma v2 est introduit avec Android 7 (Nougat) et offre une meilleure protection contre les modifications du contenu de l’APK.

L'absence de v1 indique une configuration moderne du build Android Studio.

---

## 3. Analyse du AndroidManifest.xml

### 3.1 Informations principales

- Package principal : com.example.lab4
- versionCode : 1
- versionName : 1.0
- compileSdkVersion : 36
- minSdkVersion : 24
- targetSdkVersion : 36

### Analyse

- L’application est compatible à partir d’Android 7.0 (API 24).
- Elle cible une version Android récente (API 36).
- Le compileSdk correspond à la version la plus récente utilisée pour le build.

---

## 4. Obtention des APK (Task 2)
### 4.1 Option A — APK fourni (DIVA)
- APK : `DivaApplication.apk`
-  application pédagogique vulnérable utilisée pour des tests de sécurité (analyse statique / apprentissage).

### 4.2 Option B — APK généré via Android Studio
Procédure :
- Android Studio > Build > Generate App Bundles or APKs > Build APK(s)
- Localisation : `app/build/outputs/apk/debug/app-debug.apk`
- Copie dans : `C:\APK-Analysis`

Vérifications :
- APK présent dans le dossier de travail
- signature et hash calculés
- taille notée pour référence

---

## 5. Analyse avec JADX GUI (Task 3)
APK analysé dans les captures : `app-debug.apk`

### 5.1 Ouverture dans JADX
- File > Open file...
- Sélection : `app-debug.apk`

Structure visible (panneau gauche) :
- Resources
  - `AndroidManifest.xml`
  - `res/values/strings.xml` et variantes locales (`values-xx`)
  - `classes.dex`, `classes2.dex`, `classes3.dex`, `classes4.dex`
- Source code
  - packages : `com.example.lab4` (et dépendances : androidx, kotlin, google, etc.)

---

## 6. Analyse du manifeste Android (app-debug.apk)
### 6.1 Informations principales
Extraits observés dans `AndroidManifest.xml` :
- `package="com.example.lab4"`
- `android:versionCode="1"`
- `android:versionName="1.0"`
- `android:compileSdkVersion="36"`
- `uses-sdk`
  - `android:minSdkVersion="24"`
  - `android:targetSdkVersion="36"`

Conclusion :
- Package principal : `com.example.lab4`
- Version : 1.0 (code 1)
- minSdk : 24
- targetSdk : 36
- compileSdk : 36

### 6.2 Permissions (uses-permission)
Permissions observées :
- `uses-permission android:name="com.example.lab4.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION"`

Remarque :
- Le manifeste définit également une permission custom :
  - `<permission android:name="com.example.lab4.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION" android:protectionLevel="signature" />`

Interprétation :
- permission spécifique à l’application, niveau "signature" (restreint aux apps signées avec le même certificat).

### 6.3 Attributs sensibles dans `<application>`
Attributs observés :
- `android:debuggable="true"`
- `android:allowBackup="true"`

Analyse sécurité :
- `debuggable="true"` :
  - indique un build debug, augmente les possibilités d’inspection/débogage.
  - en production, cet attribut devrait être `false`.
- `allowBackup="true"` :
  - autorise la sauvegarde/restauration potentielle des données de l’application (selon configurations et versions Android).

Aucun élément `android:usesCleartextTraffic="true"` n’a été observé dans la portion affichée.
(à confirmer en recherchant globalement dans le manifeste et les ressources).

### 6.4 Composants (activities, providers, receivers)
#### 6.4.1 Activity principale
Activité observée :
- `android:name="com.example.lab4.MainActivity"`
- `android:exported="true"`
- intent-filter :
  - action : `android.intent.action.MAIN`
  - category : `android.intent.category.LAUNCHER`

Interprétation :
- `exported="true"` est normal pour l’activité launcher (doit être accessible pour lancement par le système).

#### 6.4.2 Provider (AndroidX Startup)
Provider observé :
- `android:name="androidx.startup.InitializationProvider"`
- `android:exported="false"`
- `android:authorities="com.example.lab4.androidx-startup"`

Interprétation :
- composant non exporté, pas directement accessible aux autres applications.

#### 6.4.3 Receiver (ProfileInstaller)
Receiver observé :
- `android:name="androidx.profileinstaller.ProfileInstallReceiver"`
- `android:enabled="true"`
- `android:exported="true"`
- `android:permission="android.permission.DUMP"`
- intent-filters :
  - `androidx.profileinstaller.action.INSTALL_PROFILE`
  - `androidx.profileinstaller.action.SKIP_FILE`
  - `androidx.profileinstaller.action.SAVE_PROFILE`
  - `androidx.profileinstaller.action.BENCHMARK_OPERATION`

Analyse sécurité :
- Receiver exporté : surface d’attaque potentielle plus large.
- Présence d’une permission système `android.permission.DUMP` : limite les appels aux entités autorisées.

Note :
- Depuis Android 12, les composants avec intent-filter doivent définir explicitement `android:exported`.

---

## 7. Exploration des ressources (Task 3)
### 7.1 strings.xml
Fichier consulté :
- `res/values/strings.xml` et versions localisées (`values-af`, etc.)

Observations :
- nombreuses chaînes issues de bibliothèques (AndroidX / Material / support).
- éléments de l’UI présents.
- aucun secret évident visible dans la capture fournie (à approfondir via recherche mots-clés).

Méthode recommandée :
- rechercher dans Resources > strings.xml :
  - `password`, `secret`, `token`, `key`, `api`, `url`

### 7.2 network_security_config.xml
But :
- vérifier si un `network_security_config.xml` existe et si `cleartextTrafficPermitted` est activé.

Statut :
- non visible dans les captures fournies.
- à vérifier via recherche dans Resources (res/xml) ou Ctrl+F sur `network_security_config`.

---

## 8. Synthèse des points importants (Check Task 3)
Pour `app-debug.apk` :

- Package principal identifié : `com.example.lab4`
- Version identifiée : `versionName=1.0`, `versionCode=1`
- SDK : minSdk=24, targetSdk=36, compileSdk=36
- Permissions :
  - `com.example.lab4.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`
- Composants exportés :
  - `MainActivity` : exported=true (launcher)
  - `androidx.profileinstaller.ProfileInstallReceiver` : exported=true (avec permission DUMP)
- Configurations sensibles :
  - `android:debuggable="true"`
  - `android:allowBackup="true"`
- Ressources explorées :
  - `strings.xml` (présence de nombreuses ressources de bibliothèques)
  - network_security_config non confirmé dans les captures

---
## Diva jadx:

- Package : jakhar.aseem.diva
- versionCode : 1
- versionName : 1.0
- minSdkVersion : 15
- targetSdkVersion : 23
- platformBuildVersionCode : 23
- platformBuildVersionName : 6.0-2166767

### Analyse

L'application cible Android 6.0 (API 23) et est compatible à partir d’Android 4.0.3 (API 15).

Cela confirme qu’il s’agit d’une application ancienne, développée avant les nouvelles exigences de sécurité d’Android 7+ et Android 12.



## 1) Informations générales (AndroidManifest.xml)

- Package principal : `jakhar.aseem.diva`
- versionCode : `1`
- versionName : `1.0`
- minSdkVersion : `15`
- targetSdkVersion : `23`

## 2) Permissions demandées (uses-permission)

Permissions observées dans le manifeste :

- `android.permission.WRITE_EXTERNAL_STORAGE`
- `android.permission.READ_EXTERNAL_STORAGE`
- `android.permission.INTERNET`

## 3) Composants déclarés

### 3.1 Activities
Composants Activity identifiés (extraits visibles) :

- `jakhar.aseem.diva.MainActivity`
- `jakhar.aseem.diva.LogActivity`
- `jakhar.aseem.diva.HardcodeActivity`
- `jakhar.aseem.diva.InsecureDataStorage1Activity`
- `jakhar.aseem.diva.InsecureDataStorage2Activity`
- `jakhar.aseem.diva.InsecureDataStorage3Activity`
- `jakhar.aseem.diva.InsecureDataStorage4Activity`
- `jakhar.aseem.diva.SQLInjectionActivity`
- `jakhar.aseem.diva.InputValidation2URISchemeActivity`
- `jakhar.aseem.diva.AccessControl1Activity`
- `jakhar.aseem.diva.AccessControl2Activity`
- `jakhar.aseem.diva.AccessControl3Activity`
- `jakhar.aseem.diva.APICredsActivity`
- `jakhar.aseem.diva.APICreds2Activity`
- `jakhar.aseem.diva.Hardcode2Activity`
- `jakhar.aseem.diva.AccessControl3NotesActivity`
- `jakhar.aseem.diva.InputValidation3Activity`

Remarque : la liste complète peut être obtenue en parcourant l’ensemble du manifeste dans JADX (Ctrl+F sur `<activity`).

### 3.2 Services
Aucun service n’est visible dans les extraits fournis.
Action recommandée : rechercher `<service` dans le manifeste.

### 3.3 Receivers
Aucun receiver n’est visible dans les extraits fournis.
Action recommandée : rechercher `<receiver` dans le manifeste.

### 3.4 Providers
Provider identifié :

- `jakhar.aseem.diva.NotesProvider`
  - `android:enabled="true"`
  - `android:exported="true"`
  - `android:authorities="jakhar.aseem.diva.provider.notesprovider"`

## 4) Composants exportés / intent-filters

### 4.1 MainActivity
- Présence d’un `intent-filter` (MAIN/LAUNCHER) :
  - `android.intent.action.MAIN`
  - `android.intent.category.LAUNCHER`

Interprétation :
- Cette activité est accessible via le launcher (comportement normal pour une activité principale).
- Sur les versions Android anciennes (targetSdk 23), l’export peut être implicite si `intent-filter` présent.

### 4.2 APICredsActivity
- Présence d’un `intent-filter` personnalisé :
  - action : `jakhar.aseem.diva.action.VIEW_CREDS`
  - category : `android.intent.category.DEFAULT`

Interprétation :
- Composant potentiellement accessible depuis d’autres applications via Intent.
- Cela élargit la surface d’attaque.

### 4.3 NotesProvider (exported = true)
- `android:exported="true"` explicitement

Interprétation :
- Un provider exporté peut être interrogé par d’autres applications.
- Cela peut exposer des données ou des fonctionnalités internes si aucune protection n’est appliquée.

## 5) Configurations sensibles

### 5.1 debuggable
Dans `<application>` :
- `android:debuggable="true"`

Analyse :
- L’application est en mode debug, ce qui facilite l’inspection et le reverse engineering.
- En contexte production, ceci représente une mauvaise pratique importante.

### 5.2 usesCleartextTraffic
Aucun `android:usesCleartextTraffic="true"` n’est visible dans les extraits fournis.
Action recommandée :
- rechercher `usesCleartextTraffic` dans le manifeste (Ctrl+F), et vérifier si une configuration réseau existe.



---




# Task 6 — Comparaison JADX vs JD-GUI

## Classe analysée
`jakhar.aseem.diva.Hardcode2Activity`

Analyse réalisée à partir de :
- APK ouvert dans **JADX GUI**
- JAR généré ouvert dans **JD-GUI**

---

## Comparaison des outils

| Aspect | JADX GUI | JD-GUI |
|--------|----------|--------|
| Navigation | Affiche la structure Android complète (AndroidManifest, ressources, code Java) | Affiche uniquement la structure Java issue du JAR (packages, classes) |
| Organisation du projet | Vue orientée Android avec `res/`, `AndroidManifest.xml`, `classes.dex` | Vue classique Java avec arborescence des packages uniquement |
| Lisibilité du code | Code plus clair et plus proche du code source original | Code parfois plus brut, reconstruction moins optimisée |
| Gestion des ressources | Accès direct aux fichiers XML (strings.xml, layouts, etc.) | Aucun accès aux ressources Android |
| Gestion des annotations | Bonne préservation des annotations Android (`@Override`, etc.) | Certaines annotations peuvent être simplifiées ou perdues |
| Gestion des IDs Android | Affiche `R.layout.activity_xxx` de manière claire | Parfois montre des valeurs numériques (ex: `setContentView(2130968608)`) |
| Contexte Android | Compréhension native des composants Android (Activity, Service, Provider) | Analyse purement Java sans contexte Android global |

---

## Observation pratique sur Hardcode2Activity

### Dans JADX :
- `setContentView(R.layout.activity_hardcode2);`
- Code structuré et lisible
- Variables reconstruites proprement
- Commentaire automatique : `/* loaded from: classes.dex */`

### Dans JD-GUI :
- `setContentView(2130968608);` (ID numérique au lieu du nom de layout)
- Structure similaire mais moins contextualisée
- Moins d’informations liées aux ressources Android

---

## Forces et Faiblesses

### JADX — Forces
- Idéal pour l’analyse Android complète
- Accès au manifeste et aux ressources
- Meilleure compréhension du contexte applicatif
- Plus adapté à l’audit de sécurité mobile

### JADX — Faiblesses
- Peut être plus lourd
- Parfois reconstruction imparfaite du code complexe

---

### JD-GUI — Forces
- Simple et rapide
- Bonne lecture du bytecode Java pur
- Utile pour comparer les résultats de décompilation

### JD-GUI — Faiblesses
- Pas d’accès aux ressources Android
- Pas d’accès au manifeste
- Moins adapté aux applications Android complexes

---

