<script setup>
import axios from "axios";
</script>

<template>
  <div class="text-center position-relative">
    <div class="row justify-content-center position-absolute w-100 z-1">
      <div class="col-md-6">
        <div class="card card-body">
          <div class="input-group">
            <input
              type="text"
              class="form-control"
              placeholder="Enter address"
              :value="address"
              @input="$emit('update:address', $event.target.value)"
              ref="autocomplete"
              id="autocomplete"
            />
            <button class="btn btn-light" type="button" @click="locationButton">
              <span
                class="spinner-grow spinner-grow-sm"
                :class="{ 'visually-hidden': !spinner }"
                aria-hidden="true"
              ></span>
              <span :class="{ 'visually-hidden': spinner }" role="status">
                <i class="bi bi-crosshair"></i
              ></span>
            </button>
            <button
              class="btn btn-primary"
              type="button"
              @click="searchLocation"
            >
              <i class="bi bi-search"></i>
            </button>
          </div>
          <div v-if="alert[0]" class="mt-3">
            <div
              :class="['alert', 'alert-' + alert[2], 'alert-dismissible']"
              role="alert"
            >
              <div>{{ alert[1] }}</div>
              <button
                type="button"
                class="btn-close"
                @click="closeAlert"
                aria-label="Close"
              ></button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    address: String,
    alert: Array,
    addSearchToHistory: Function,
    searchLocation: Function,
    closeAlert: Function,
    showAlert: Function,
  },
  data() {
    return {
      spinner: false,
      center: { lat: 43.653226, lng: -79.3720226 },
    };
  },
  methods() {
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
          this.addSearchToHistory(results[0].formatted_address, this.center);
        } else {
          this.searchLocation();
        }
      });
    }
  },
  methods: {
    locationButton() {
      this.spinner = true;
      const apiKey = import.meta.env.VITE_GOOGLE_API_KEY;

      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            //find address
            axios
              .get(
                "https://maps.googleapis.com/maps/api/geocode/json?latlng=" +
                  position.coords.latitude +
                  "," +
                  position.coords.longitude +
                  "&key=" +
                  apiKey
              )
              .then((response) => {
                if (response.data.error_message) {
                  alertTrigger.addEventListener("click", () => {
                    this.showAlert(response.data.error_message, "warning");
                  });
                } else {
                  this.$emit(
                    "update:address",
                    response.data.results[0].formatted_address
                  );
                  this.center = {
                    lat: position.coords.latitude,
                    lng: position.coords.longitude,
                  };
                  this.addSearchToHistory(
                    response.data.results[0].formatted_address,
                    this.center
                  );
                }
                this.spinner = false;
              });
          },
          (error) => {
            this.showAlert(error.messgae, "warning");
            console.error();
          }
        );
      } else {
        this.showAlert(
          "your browser dose not support geolocation API",
          "warning"
        );
        this.spinner = false;
      }
    },
  },
};
</script>
