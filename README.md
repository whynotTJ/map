# Interaktive Karten-App (Android & Web)

Eine moderne, interaktive Karten-Applikation entwickelt mit **Vue 3**, **Vite**, **TypeScript**, **Leaflet** und **Capacitor 8**.
Die Anwendung ist sowohl als native Android-App als auch als responsive Web-Applikation im Browser lauffähig.

---

## 🚀 Features

- **🗺️ Interaktive Karte:** Flüssiges Verschieben (Pan) und Zoomen mit Touch- und Maus-Gesten auf Basis von OpenStreetMap (`@vue-leaflet/vue-leaflet`).
- **📍 GPS-Standortbestimmung:** 
  - **Android:** Native Hardware-Ortung über `@capacitor/geolocation` inklusive Berechtigungsprüfung und Direktverlinkung zu den Android-Systemeinstellungen bei Verweigerung (`@capawesome/capacitor-settings-launcher`).
  - **Web:** Automatischer Fallback auf die HTML5 Geolocation API (`navigator.geolocation`).
- **🔍 Lokale Adresssuche (Forward Geocoding):**
  - **Android:** Rein gerätelokales Geocoding über die native Android-Schnittstelle (`android.location.Geocoder`) mithilfe von `@capgo/capacitor-nativegeocoder` – ohne externe API-Keys oder Kontenzwang.
  - **Web:** Fallback zur OpenStreetMap Nominatim REST-API.
- **🔴🔵 Visuell getrennte Kreismarker:** Verwendung von performanten, SVG-basierten `<l-circle-marker>`-Komponenten:
  - **Roter Marker:** Gefundene Adresse aus der Suchleiste.
  - **Blauer Marker:** Eigener GPS-Standort.
  - Beide Marker können zeitgleich auf der Karte dargestellt werden.
- **💾 Persistente Zustandsspeicherung:** Speichert den aktuellen Mittelpunkt und die Zoomstufe über `@capacitor/preferences` (Android `SharedPreferences` / Web `localStorage`), sodass der vorherige Zustand beim Neustart exakt wiederhergestellt wird.
- **⏳ Ladeindikatoren & Feedback:** Modernes Glassmorphismus-Lade-Overlay bei asynchronen Operationen sowie Toasts bei Erfolgs- und Fehlermeldungen.

---

## 🛠️ Voraussetzungen & Installation

### 1. Repository klonen & Abhängigkeiten installieren

```bash
git clone https://github.com/whynotTJ/map.git
cd map
npm install
```

---

## 💻 Entwicklung & Ausführung

### Web-Anwendung (Lokaler Dev-Server)

Um die Web-Applikation im Browser zu testen:

```bash
npm run dev
```
Die Anwendung ist anschließend unter **[http://localhost:5173/](http://localhost:5173/)** erreichbar.

---

### Android (Nativ)

1. **Web-Assets bauen & mit Android synchronisieren:**
   ```bash
   npm run build
   npx cap sync android
   ```

2. **In Android Studio öffnen:**
   ```bash
   npx cap open android
   ```

3. **Alternativ direkt über das Terminal kompilieren:**
   ```bash
   cd android
   ./gradlew assembleDebug
   ```
   *(Unter Windows PowerShell: `.\gradlew.bat assembleDebug`)*

Die generierte APK befindet sich anschließend unter:  
`android/app/build/outputs/apk/debug/app-debug.apk`

---

## 🏗️ Verwendete Technologien & Plugins

- **Frontend:** [Vue 3](https://vuejs.org/) (Composition API), [TypeScript](https://www.typescriptlang.org/), [Vite](https://vitejs.dev/)
- **Karten-Framework:** [Leaflet](https://leafletjs.com/), [@vue-leaflet/vue-leaflet](https://github.com/vue-leaflet/vue-leaflet), OpenStreetMap Tiles
- **Mobile Bridge:** [Capacitor 8](https://capacitorjs.com/)
- **Plugins:**
  - `@capacitor/geolocation` – GPS-Standort
  - `@capacitor/preferences` – Zustandspersistierung
  - `@capgo/capacitor-nativegeocoder` – Lokales Android Geocoding
  - `@capawesome/capacitor-settings-launcher` – App-Settings Intent

