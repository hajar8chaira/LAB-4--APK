# Lab AB4 — Analyse statique d’un APK avec JADX GUI  
## Task 3 — Analyse du AndroidManifest et des ressources

---

## APK analysé

- Nom du fichier : `UnCrackable-Level1.apk`
- Source : OWASP MSTG Crackme (fourni par l’enseignant)
- Outil utilisé : JADX GUI

---

# 1. Analyse du AndroidManifest.xml

Fichier analysé :  
`Resources → AndroidManifest.xml`

---

## Informations générales

| Élément | Valeur |
|----------|--------|
| Package | `owasp.mstg.uncrackable1` |
| versionCode | `1` |
| versionName | `1.0` |
| minSdkVersion | `19` (Android 4.4) |
| targetSdkVersion | `28` (Android 9) |

---

# 2. Permissions demandées

Recherche des balises :

```xml
<uses-permission>
```

## Résultat

Aucune permission n’est déclarée dans le manifeste.

## Conclusion Sécurité

L’application ne demande aucune permission système :
- Internet  
- Stockage  
- Caméra  
- Localisation  

La surface d’attaque liée aux permissions est minimale.

---

# 3. Composants déclarés

## Activity principale

```xml
<activity
    android:name="sg.vantagepoint.uncrackable1.MainActivity">
```

Cette activité contient :

```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN"/>
    <category android:name="android.intent.category.LAUNCHER"/>
</intent-filter>
```

## Analyse

- `MainActivity` est l’activité principale.
- Elle est lancée au démarrage de l’application.
- Aucun autre composant n’est déclaré :
  - Service  
  - BroadcastReceiver  
  - ContentProvider  

L’application est mono-activity.

---

# 4. Composants exportés

L’attribut `android:exported` n’est pas explicitement défini.

Cependant :

- La présence d’un `intent-filter`
- Avec `MAIN` et `LAUNCHER`
- Implique que l’activité soit exportée implicitement (comportement Android < 12)

## Impact Sécurité

| Élément | Risque |
|----------|--------|
| MainActivity exportée implicitement | Faible (comportement normal pour une application launcher) |

---

# 5. Configurations sensibles

Attributs vérifiés :

- `android:debuggable`
- `android:usesCleartextTraffic`
- `android:allowBackup`

## Résultats

| Attribut | Valeur | Analyse |
|-----------|--------|----------|
| debuggable | Non présent | Mode debug non activé |
| usesCleartextTraffic | Non présent | Pas d’autorisation HTTP explicite |
| allowBackup | true | Sauvegarde autorisée |

## Analyse Sécurité

`android:allowBackup="true"` permet :

- Sauvegarde des données via ADB
- Extraction potentielle des données locales

Cela représente un risque moyen si des données sensibles sont stockées.

---

# 6. Analyse des ressources (strings.xml)

Fichier analysé :  
`res/values/strings.xml`

## Chaînes identifiées

```xml
<string name="edit_text">Enter the Secret String</string>
<string name="button_verify">Verify</string>
```

## Interprétation

- L’application attend une Secret String.
- Cela indique une logique de vérification interne.
- Aucun secret hardcodé visible.
- Aucune clé API détectée.

---

¡
