<script setup>
import SearchHistory from "@/components/SearchHistory.vue";
import Search from "@/components/Search.vue";
import { toRaw } from "vue";
import axios from "axios";
</script>

<template>
  <div class="row vh-100">
    <div id="sidebarMenu" class="col-3 p-0">
      <SearchHistory
        :searchHistory="searchHistory"
        :goToLocation="goToLocation"
        @update:searchHistory="updateSearchHistory"
      />
    </div>
    <div class="col-9 p-0">
      <Search
        :address="address"
        @update:address="updateAddress"
        :alert="alert"
        :searchLocation="searchLocation"
        :addSearchToHistory="addSearchToHistory"
        :closeAlert="closeAlert"
        :showAlert="showAlert"
      />
      <div id="map" class="h-100 bg-light"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      alert: [false, "", "success"],
      address: "",
      center: { lat: 43.653226, lng: -79.3720226 },
      searchHistory: [],
    };
  },
  mounted() {
    if (window.google && window.google.maps) {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((position) => {
          this.center = {
            lat: position.coords.latitude,
            lng: position.coords.longitude,
          };
        });
      }
      this.initMap();
    }
    if (window.google && window.google.maps && window.google.maps.places) {
      let autocomplete = new google.maps.places.Autocomplete(
        document.getElementById("autocomplete")
      );

      autocomplete.addListener("place_changed", () => {
        let place = autocomplete.getPlace();
        if (place.formatted_address) {
          this.address = place.formatted_address;
          this.center = {
            lat: place.geometry.location.lat(),
            lng: place.geometry.location.lng(),
          };
          this.addSearchToHistory(place.formatted_address, this.center);
        } else {
          this.searchLocation();
        }
      });
    }
  },
  methods: {
    initMap() {
      const mapOptions = {
        zoom: 15,
        center: this.center,
        mapTypeId: google.maps.MapTypeId.ROADMAP,
      };
      this.map = new google.maps.Map(
        document.getElementById("map"),
        mapOptions
      );

      this.marker = new google.maps.Marker({
        position: this.center,
        map: this.map,
      });
    },
    showAlert(message, type) {
      this.alert = [true, message, type];
    },
    closeAlert() {
      this.alert = [false, "", "success"];
    },
    searchLocation() {
      if (!this.address) {
        this.showAlert("Address is empty", "warning");
        return;
      }
      const placesService = new google.maps.places.PlacesService(this.map);

      placesService.textSearch({ query: this.address }, (results, status) => {
        if (status === google.maps.places.PlacesServiceStatus.OK) {
          this.center = {
            lat: results[0].geometry.location.lat(),
            lng: results[0].geometry.location.lng(),
          };
          this.addSearchToHistory(results[0].formatted_address, this.center);
        } else {
          this.showAlert("Places search failed", "warning");
        }
      });
    },
    addMarker(position) {
      const marker = new google.maps.Marker({
        position: position,
        map: this.map,
      });
      return marker;
    },
    updateAddress(newAddress) {
      this.address = newAddress;
    },
    goToLocation(position) {
      this.map.setCenter(position);
    },
    addSearchToHistory(address, position) {
      const exists = this.searchHistory.some(
        (item) =>
          item.position.lat === position.lat &&
          item.position.lng === position.lng
      );

      if (!exists) {
        this.goToLocation(position);
        const marker = this.addMarker(position);
        const apiKey = import.meta.env.VITE_GOOGLE_API_KEY;

        axios
          .get(
            "https://maps.googleapis.com/maps/api/timezone/json?location=" +
              position.lat +
              "," +
              position.lng +
              "&timestamp=" +
              Math.floor(Date.now() / 1000) +
              "&key=" +
              apiKey
          )
          .then((timezoneResponse) => {
            const { timeZoneId, dstOffset, rawOffset } = timezoneResponse.data;
            const currentTime = new Date();
            const utcTime =
              currentTime.getTime() + currentTime.getTimezoneOffset() * 60000;
            const localTime = new Date(
              utcTime + (rawOffset + dstOffset) * 1000
            );

            this.searchHistory.push({
              address,
              position,
              marker,
              timeZone: timeZoneId,
              localTime: new Intl.DateTimeFormat("en-US", {
                timeZone: timeZoneId,
                hour: "numeric",
                minute: "numeric",
                second: "numeric",
                hour12: true,
              }).format(localTime),
            });
          })
          .catch((error) => {
            console.error("Error fetching time zone:", error);
          });
      }
    },
    updateSearchHistory(newSearchHistory) {
      const deletedItems = this.searchHistory.filter(
        (item) => !newSearchHistory.includes(item)
      );
      deletedItems.forEach((item) => {
        toRaw(item.marker).setMap(null);
      });
      this.searchHistory = newSearchHistory;
    },
  },
};
</script>

<style scoped></style>
