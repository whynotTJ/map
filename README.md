# Karten-App (Android & Web)

Eine interaktive Karten-Applikation entwickelt mit Vue 3, Vite, TypeScript, Leaflet und Capacitor 8.
Unterstützt sowohl nativen Ausführung auf Android als auch einen Web-Browser Fallback für Präsentationszwecke.

## Features

- 🗺️ **Interaktive Karte:** Zoom, Pan und Verschiebungen mit Touch- und Maus-Gesten (OpenStreetMap).
- 📍 **GPS-Standortbestimmung:** 
  - **Android:** Nutzung der nativen Geräte-Hardware über `@capacitor/geolocation` (inklusive automatischer Berechtigungsauswertung & Einstellungen-Link bei Ablehnung).
  - **Web:** Nahtloser Fallback auf die HTML5 Browser Geolocation API (`navigator.geolocation`).
- 🔍 **Adresssuche (Geocoding):**
  - **Android:** Geocoding via `@capawesome-team/capacitor-geocoder`.
  - **Web:** Automatisierter Fallback zur OpenStreetMap Nominatim REST-API.
- 💾 **Persistente Zustandsspeicherung:** Karte behält die letzte Zoomstufe und Mittelpunkts-Koordinaten über `@capacitor/preferences` (SharedPreferences / LocalStorage).
- 🎨 **Modernes UI:** Glassmorphismus-Suchleiste, animierter GPS-Puls-Marker und Toast-Benachrichtigungen.

## Voraussetzungen & Setup

### 1. Capawesome Plugin Lizenzschlüssel (`.npmrc`)

Das Projekt nutzt das private Plugin `@capawesome-team/capacitor-geocoder`. Da der Lizenzschlüssel aus Sicherheitsgründen nicht in Git publiziert wird (`.npmrc` ist durch die `.gitignore` komplett gesperrt!), muss im Hauptverzeichnis des Projekts vor dem Installieren manuell eine Datei namens `.npmrc` angelegt werden:

```ini
@capawesome-team:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=DEIN_GEHEIMER_LIZENZSCHLUESSEL_HIER
```
*(Hinweis: Ersetze `DEIN_GEHEIMER_LIZENZSCHLUESSEL_HIER` durch den entsprechenden Capawesome Token).*

### 2. Abhängigkeiten installieren

```bash
npm install
```

### 3. Web-Anwendung (Lokaler Server) starten

```bash
npm run dev
```
Die Webanwendung läuft anschließend lokal unter [http://localhost:5173/](http://localhost:5173/).

### 4. Nativ für Android kompilieren & ausführen

```bash
npm run build
npx cap sync android
```

Danach kann die App in Android Studio geöffnet oder im Emulator gebaut werden:
```bash
npx cap open android
# oder alternativ direkt über Gradle im Terminal:
cd android
./gradlew assembleDebug
```
