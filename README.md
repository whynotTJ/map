# Interaktive Karten-App (Android & Web)

Eine interaktive Karten-Applikation entwickelt mit Vue 3, TypeScript, Leaflet und Capacitor 8. Die Anwendung kann sowohl als native Android-App als auch als Web-Applikation im Browser ausgeführt werden.

---

## Features

- **Interaktive Karte:** Verschieben und Zoomen mit Touch- und Maus-Gesten auf Basis von OpenStreetMap (`@vue-leaflet/vue-leaflet`).
- **GPS-Standortbestimmung:** 
  - **Android:** Native Ortung über `@capacitor/geolocation` inklusive Berechtigungsabfrage und Verlinkung zu den Systemeinstellungen bei Verweigerung (`@capawesome/capacitor-settings-launcher`).
  - **Web:** Fallback auf die HTML5 Geolocation API (`navigator.geolocation`).
- **Lokale Adresssuche (Forward Geocoding):**
  - **Android:** Gerätelokales Geocoding über die Android-Schnittstelle (`android.location.Geocoder`) via `@capgo/capacitor-nativegeocoder` (ohne externe API-Schlüssel).
  - **Web:** Fallback zur OpenStreetMap Nominatim REST-API.
- **Getrennte Kreismarker:** Verwendung von `<l-circle-marker>`-Komponenten:
  - Roter Marker für gefundene Adressen aus der Suche.
  - Blauer Marker für den aktuellen GPS-Standort.
  - Beide Marker können zeitgleich angezeigt werden.
- **Persistente Zustandsspeicherung:** Speichert Kartenzentrum und Zoomstufe über `@capacitor/preferences` (Android `SharedPreferences` / Web `localStorage`), sodass der Zustand nach einem Neustart erhalten bleibt.
- **Ladeindikatoren & Feedback:** Visuelle Rückmeldung bei laufender Ortung oder Suche sowie Statusmeldungen (Toasts).

---

## Installation & Setup

### 1. Repository klonen und Abhängigkeiten installieren

```bash
git clone https://github.com/whynotTJ/map.git
cd map
npm install
```

---

## Ausführung & Build

### Web-Anwendung (Lokaler Dev-Server)

```bash
npm run dev
```
Die Anwendung ist standardmäßig unter `http://localhost:5173/` erreichbar.

---

### Android (Nativ)

1. Web-Assets bauen und mit Android synchronisieren:
   ```bash
   npm run build
   npx cap sync android
   ```

2. Projekt in Android Studio öffnen:
   ```bash
   npx cap open android
   ```

3. Alternativ direkt über das Terminal kompilieren:
   ```bash
   cd android
   ./gradlew assembleDebug
   ```
   *(Windows PowerShell: `.\gradlew.bat assembleDebug`)*

Die generierte APK befindet sich unter:  
`android/app/build/outputs/apk/debug/app-debug.apk`

---

## Verwendete Technologien & Bibliotheken

- **Frontend:** Vue 3 (Composition API), TypeScript, Vite
- **Karten:** Leaflet, @vue-leaflet/vue-leaflet, OpenStreetMap
- **Mobile Bridge:** Capacitor 8
- **Plugins:**
  - `@capacitor/geolocation`
  - `@capacitor/preferences`
  - `@capgo/capacitor-nativegeocoder`
  - `@capawesome/capacitor-settings-launcher`


