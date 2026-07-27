<script setup lang="ts">
import { ref } from "vue";
import { LMap, LTileLayer, LMarker, LIcon } from "@vue-leaflet/vue-leaflet";
import "leaflet/dist/leaflet.css";
import L from "leaflet";

import { Capacitor } from "@capacitor/core";
import { Geolocation } from "@capacitor/geolocation";
import { Preferences } from "@capacitor/preferences";
import { Geocoder } from "@capawesome-team/capacitor-geocoder";
import { SettingsLauncher } from "@capawesome/capacitor-settings-launcher";

import markerIcon from "leaflet/dist/images/marker-icon.png";
import markerIconRetina from "leaflet/dist/images/marker-icon-2x.png";
import markerShadow from "leaflet/dist/images/marker-shadow.png";

L.Icon.Default.mergeOptions({
  iconRetinaUrl: markerIconRetina,
  iconUrl: markerIcon,
  shadowUrl: markerShadow,
});

const initialZoom = ref(13);
const initialCenter = ref<[number, number]>([52.520008, 13.404954]); //Berlin als Startort

const searchQuery = ref("");
const searchMarker = ref<[number, number] | null>(null);

const userLocation = ref<[number, number] | null>(null);
const isGpsActive = ref(false);

const isLoading = ref(false);
const loadingMessage = ref("");

interface Toast {
  id: number;
  title: string;
  message: string;
  type: "success" | "error" | "permission-denied";
}
const toasts = ref<Toast[]>([]);
let nextToastId = 0;

const showToast = (toast: Omit<Toast, "id">) => {
  const id = nextToastId++;
  toasts.value.push({ ...toast, id });
  if (toast.type !== "permission-denied") {
    setTimeout(() => {
      removeToast(id);
    }, 6000);
  }
};

const removeToast = (id: number) => {
  toasts.value = toasts.value.filter((t) => t.id !== id);
};

const map = ref<any>(null);
const mapInstance = ref<L.Map | null>(null);

const onMapReady = async (leafletMap: L.Map) => {
  mapInstance.value = leafletMap;

  try {
    const { value: savedCenter } = await Preferences.get({ key: "map-center" });
    const { value: savedZoom } = await Preferences.get({ key: "map-zoom" });

    if (savedCenter) {
      const parsed = JSON.parse(savedCenter);
      if (Array.isArray(parsed) && parsed.length === 2) {
        const targetZoom = savedZoom ? parseInt(savedZoom, 10) : 13;
        leafletMap.setView([parsed[0], parsed[1]], targetZoom, { animate: false });
      }
    }
  } catch (e) {
    console.error("Failed to restore map state in onMapReady", e);
  }

  leafletMap.on("moveend", async () => {
    const newCenter = leafletMap.getCenter();
    const newZoom = leafletMap.getZoom();

    try {
      await Preferences.set({
        key: "map-center",
        value: JSON.stringify([newCenter.lat, newCenter.lng]),
      });
      await Preferences.set({
        key: "map-zoom",
        value: newZoom.toString(),
      });
    } catch (e) {
      console.error("Failed to persist map state on moveend", e);
    }
  });
};

const updateMapPosition = (lat: number, lng: number, targetZoom?: number) => {
  const targetMap = mapInstance.value || (map.value && map.value.leafletObject);
  const newZoom = targetZoom !== undefined ? targetZoom : (targetMap ? targetMap.getZoom() : 15);

  if (targetMap) {
    targetMap.setView([lat, lng], newZoom, { animate: true });
  }
};

const locateUser = async () => {
  try {
    isLoading.value = true;
    loadingMessage.value = "Position wird per GPS bestimmt...";

    let lat: number;
    let lng: number;
    let accuracy = 0;

    if (Capacitor.getPlatform() === "web") {
      if (!navigator.geolocation) {
        showToast({
          title: "Ortung nicht unterstützt",
          message: "Ihr Browser unterstützt keine Standortbestimmung.",
          type: "error",
        });
        return;
      }
      const position = await new Promise<GeolocationPosition>((resolve, reject) => {
        navigator.geolocation.getCurrentPosition(resolve, reject, {
          enableHighAccuracy: true,
          timeout: 10000,
        });
      });
      lat = position.coords.latitude;
      lng = position.coords.longitude;
      accuracy = position.coords.accuracy || 0;
    } else {
      let permStatus = await Geolocation.checkPermissions();
      if (permStatus.location !== "granted") {
        permStatus = await Geolocation.requestPermissions();
        if (permStatus.location !== "granted") {
          showToast({
            title: "Standortberechtigung verweigert",
            message: "Der Zugriff auf Ihren Standort wurde verweigert. Sie können dies in den App-Einstellungen aktivieren.",
            type: "permission-denied",
          });
          return;
        }
      }

      const position = await Geolocation.getCurrentPosition({
        enableHighAccuracy: true,
        timeout: 10000,
      });

      lat = position.coords.latitude;
      lng = position.coords.longitude;
      accuracy = position.coords.accuracy || 0;
    }

    userLocation.value = [lat, lng];
    updateMapPosition(lat, lng, 16);
    isGpsActive.value = true;

    showToast({
      title: "Position erfolgreich bestimmt",
      message: `Genauigkeit auf ca. ${Math.round(accuracy)} Meter.`,
      type: "success",
    });
  } catch (error: any) {
    console.error("Geolocation error:", error);
    let errMsg = error.message || "Standort konnte nicht bestimmt werden.";
    if (error.code === 1 || errMsg.toLowerCase().includes("denied")) {
      showToast({
        title: "Standortberechtigung verweigert",
        message: "Der Standortzugriff wurde blockiert oder verweigert.",
        type: "permission-denied",
      });
    } else if (
      errMsg.toLowerCase().includes("location disabled") ||
      errMsg.toLowerCase().includes("gps disabled") ||
      error.code === 2
    ) {
      showToast({
        title: "GPS deaktiviert",
        message: "Das GPS auf Ihrem Gerät ist deaktiviert. Bitte aktivieren Sie es in den Systemeinstellungen.",
        type: "error",
      });
    } else {
      showToast({
        title: "Ortungsfehler",
        message: "Verbindung fehlgeschlagen oder GPS-Signal zu schwach. " + errMsg,
        type: "error",
      });
    }
  } finally {
    isLoading.value = false;
  }
};

const searchAddress = async () => {
  const query = searchQuery.value.trim();
  if (!query) return;

  if (!navigator.onLine) {
    showToast({
      title: "Keine Internetverbindung",
      message: "Bitte verbinden Sie sich mit dem Internet, um nach Adressen zu suchen.",
      type: "error",
    });
    return;
  }

  try {
    isLoading.value = true;
    loadingMessage.value = `Suche nach "${query}"...`;

    let lat: number | null = null;
    let lng: number | null = null;

    if (Capacitor.getPlatform() !== "web") {
      try {
        const result = await Geocoder.geocode({
          address: query,
        });
        if (result && typeof result.latitude === "number" && typeof result.longitude === "number") {
          lat = result.latitude;
          lng = result.longitude;
        }
      } catch (pluginError) {
        // Silently continue to Nominatim fallback if native geocoding service is offline/unavailable
      }
    }

    if (lat === null || lng === null) {
      const response = await fetch(
        `https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(query)}&limit=1`,
        {
          headers: {
            "Accept-Language": "de,en",
          },
        }
      );
      const data = await response.json();
      if (Array.isArray(data) && data.length > 0) {
        lat = parseFloat(data[0].lat);
        lng = parseFloat(data[0].lon);
      }
    }

    if (lat !== null && lng !== null && !isNaN(lat) && !isNaN(lng)) {
      searchMarker.value = [lat, lng];
      updateMapPosition(lat, lng, 15);

      showToast({
        title: "Adresse gefunden",
        message: `Die Adresse wurde auf der Karte markiert.`,
        type: "success",
      });
    } else {
      showToast({
        title: "Adresse nicht gefunden",
        message: `Die Adresse "${query}" konnte nicht gefunden werden.`,
        type: "error",
      });
    }
  } catch (error: any) {
    console.error("Geocoding error:", error);
    showToast({
      title: "Suche fehlgeschlagen",
      message: error.message || "Es trat ein Fehler bei der Adresssuche auf.",
      type: "error",
    });
  } finally {
    isLoading.value = false;
  }
};

const openAppSettings = async () => {
  try {
    if (Capacitor.getPlatform() === "web") {
      showToast({
        title: "Browser-Einstellungen",
        message: "Bitte aktivieren Sie die Standortberechtigung in den Einstellungen Ihres Web-Browsers.",
        type: "error",
      });
      return;
    }
    await SettingsLauncher.openAppSettings();
  } catch (error: any) {
    console.error("Settings Launcher error:", error);
    showToast({
      title: "Fehler beim Öffnen der Einstellungen",
      message: "Bitte öffnen Sie die Einstellungen manuell über die Systemeinstellungen.",
      type: "error",
    });
  }
};

const mapOptions = {
  zoomControl: true,
  dragging: true,
  touchZoom: true,
  doubleClickZoom: true,
  scrollWheelZoom: true,
  boxZoom: true,
  tap: false,
};

</script>

<template>
  <div id="app">
    <!-- Map Canvas -->
    <div class="map-wrapper">
      <l-map
        ref="map"
        :zoom="initialZoom"
        :center="initialCenter"
        :use-global-leaflet="false"
        :options="mapOptions"
        class="map-container"
        @ready="onMapReady"
      >
        <l-tile-layer
          url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
          layer-type="base"
          name="OpenStreetMap"
        ></l-tile-layer>

        <!-- Search Marker -->
        <l-marker v-if="searchMarker" :lat-lng="searchMarker"></l-marker>

        <!-- User Current Location Marker  -->
        <l-marker v-if="userLocation" :lat-lng="userLocation">
          <l-icon
            :icon-size="[32, 32]"
            :icon-anchor="[16, 16]"
            class-name="user-location-marker"
          >
            <div class="user-location-pulse"></div>
            <div class="user-location-dot"></div>
          </l-icon>
        </l-marker>
      </l-map>
    </div>

    <!-- Floating UI Controls -->
    <div class="floating-ui">
      <!-- Top Section: Search Bar -->
      <div class="search-container">
        <input
          v-model="searchQuery"
          type="text"
          class="search-input"
          placeholder="Nach Adresse suchen..."
          @keyup.enter="searchAddress"
        />
        <button class="search-button" @click="searchAddress">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            height="18"
            viewBox="0 -960 960 960"
            width="18"
            fill="currentColor"
          >
            <path
              d="M784-120 532-372q-30 24-74 38t-90 14q-117 0-198.5-81.5T88-600q0-117 81.5-198.5T368-880q117 0 198.5 81.5T648-600q0 46-14 90t-38 74l252 252-64 64ZM368-292q70 0 119-49t49-119q0-70-49-119t-119-49q-70 0-119 49t-49 119q0 70 49 119t119 49Z"
            />
          </svg>
          Suchen
        </button>
      </div>

      <!-- Bottom Right Section: GPS Button -->
      <div class="action-buttons">
        <button
          class="gps-button"
          :class="{ loading: isLoading && loadingMessage.includes('GPS'), active: isGpsActive }"
          title="Meinen Standort bestimmen"
          @click="locateUser"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            height="24"
            viewBox="0 -960 960 960"
            width="24"
          >
            <path
              d="M440-122v-82q-114-15-192-93t-93-192h-82v-80h82q15-114 93-192t192-93v-82h80v82q114 15 192 93t93 192h82v80h-82q-15 114-93 192t-192 93v82h-80Zm40-158q92 0 156-64t64-156q0-92-64-156t-156-64q-92 0-156 64t-64 156q0 92 64 156t156 64Zm0-120q-42 0-71-29t-29-71q0-42 29-71t71-29q42 0 71 29t29 71q0 42-29 71t-71 29Z"
            />
          </svg>
        </button>
      </div>
    </div>

    <!-- Glassmorphic Loading Overlay -->
    <div v-if="isLoading" class="loading-overlay">
      <div class="spinner"></div>
      <div class="loading-text">{{ loadingMessage }}</div>
    </div>

    <!-- Alert / Toast Messages -->
    <div class="toast-container">
      <div
        v-for="toast in toasts"
        :key="toast.id"
        class="toast"
      >
        <div class="toast-content">
          <span class="toast-title">{{ toast.title }}</span>
          <span class="toast-message">{{ toast.message }}</span>
        </div>
        <div class="toast-actions">
          <button
            v-if="toast.type === 'permission-denied'"
            class="toast-button"
            @click="openAppSettings"
          >
            Einstellungen öffnen
          </button>
          <button class="toast-close" @click="removeToast(toast.id)">
            &times;
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
