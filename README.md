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

## Voraussetzungen

Für einen reproduzierbaren Android-Build werden benötigt:

- Git
- Node.js 22 oder neuer
- JDK 17 oder neuer
- Android Studio
- Android SDK Platform 36
- Ein Android-Emulator mit API 35 oder 36 oder ein verbundenes Android-Gerät
- Eine Internetverbindung für die erstmaligen Downloads sowie für Kartenkacheln und Geocoding

Die installierten Versionen können im Terminal geprüft werden:

```powershell
node --version
java -version
```

## Installation aus einem frischen Klon

Alle folgenden Befehle müssen im Projektstamm ausgeführt werden. Der Projektstamm ist der Ordner, der die Dateien `package.json` und `capacitor.config.ts` enthält.

```powershell
git clone https://github.com/whynotTJ/map.git
cd map
npm ci
npm run build
npx cap sync android
npx cap open android
```

Beim ersten Öffnen in Android Studio:

1. Den Gradle-Sync vollständig abwarten.
2. Von Android Studio vorgeschlagene SDK-Komponenten installieren.
3. Unter `Tools > Device Manager` einen Emulator mit API 35 oder 36 erstellen und starten.
4. Den gestarteten Emulator oben in der Geräteliste auswählen.
5. Über den grünen Run-Button die Konfiguration `app` starten.

Android Studio erzeugt dabei die lokale Datei `android/local.properties` mit dem SDK-Pfad. Diese Datei ist rechnerabhängig und wird deshalb nicht im Repository gespeichert.

## Erneuter Android-Start

Nach Änderungen am Vue-Code müssen die Web-Assets erneut gebaut und synchronisiert werden:

```powershell
npm run build
npx cap sync android
```

Danach wird die App erneut über den Run-Button in Android Studio gestartet.

## APK über das Terminal erstellen

Nach der erstmaligen Einrichtung durch Android Studio kann die Debug-APK unter Windows auch direkt gebaut werden:

```powershell
cd android
.\gradlew.bat assembleDebug
cd ..
```

Die erzeugte APK befindet sich unter:

`android/app/build/outputs/apk/debug/app-debug.apk`

## Standort im Emulator testen

1. Im laufenden Emulator die erweiterten Einstellungen öffnen.
2. Unter `Location` einen Standort auswählen und an das Gerät senden.
3. In der App die Standortberechtigung erlauben.
4. Die Schaltfläche für den aktuellen Standort betätigen.

## Häufige Fehler

### Android-Plattform angeblich nicht vorhanden

Wenn Capacitor meldet, dass die Android-Plattform nicht hinzugefügt wurde, befindet sich das Terminal meistens im Unterordner `android`. Mit `cd ..` in den Projektstamm wechseln und den Befehl dort erneut ausführen. `npx cap` darf nicht aus dem Unterordner `android` gestartet werden.

### Karte bleibt leer

Für die OpenStreetMap-Kartenkacheln wird eine aktive Internetverbindung benötigt.

## Web-Anwendung

```powershell
npm run dev
```

Die Anwendung ist standardmäßig unter `http://localhost:5173/` erreichbar.

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


