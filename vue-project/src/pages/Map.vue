<script setup>
import axios from "axios";
</script>

<template>
  <!-- <button @click="locationButton" class="btn">Your Location</button> -->
  <div class="container text-center">
    <div class="row justify-content-center position-relative z-1">
      <div class="col-md-6">
        <div class="card card-body">
          <div class="input-group mb-3">
            <input
              type="text"
              class="form-control"
              placeholder="Enter address"
              v-model="address"
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
        </div>
      </div>
    </div>
    <div
      id="map"
      class="position-absolute top-0 start-0 bottom-0 end-0 bg-light"
    ></div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      address: "",
      spinner: false,
      center: { lat: 0, lng: 0 },
      markers: [{}],
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
        console.log(place.formatted_address);
        if (place.formatted_address) {
          this.address = place.formatted_address;
          this.center = {
            lat: place.geometry.location.lat(),
            lng: place.geometry.location.lng(),
          };
          this.map.setCenter(this.center);
          this.marker.setPosition(this.center);
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
    locationButton() {
      this.spinner = true;

      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            this.getAddress(
              position.coords.latitude,
              position.coords.longitude
            );
          },
          (error) => {
            console.error(error.messgae);
          }
        );
      } else {
        console.log("your browser dose not support geolocation API");
        this.spinner = false;
      }
    },
    getAddress(latitude, longitude) {
      axios
        .get(
          "https://maps.googleapis.com/maps/api/geocode/json?latlng=" +
            latitude +
            "," +
            longitude +
            "&key=AIzaSyD56S9OgTswBWjju-I2JJ16YwtDQN8RZOk"
        )
        .then((response) => {
          if (response.data.error_message) {
            console.log(response.data.error_message);
          } else {
            this.address = response.data.results[0].formatted_address;
            this.center = { lat: latitude, lng: longitude };
            this.map.setCenter(this.center);
            this.marker.setPosition(this.center);
          }
          this.spinner = false;
        });
    },
    searchLocation() {
      if (!this.address) {
        console.log("no add");
        return;
      }
      const placesService = new google.maps.places.PlacesService(this.map);

      placesService.textSearch({ query: this.address }, (results, status) => {
        if (status === google.maps.places.PlacesServiceStatus.OK) {
          this.center = {
            lat: results[0].geometry.location.lat(),
            lng: results[0].geometry.location.lng(),
          };
          this.map.setCenter(this.center);
          this.marker.setPosition(this.center);
        } else {
          console.error("Places search failed:", status);
        }
      });
    },
  },
};
</script>

<style scoped></style>
