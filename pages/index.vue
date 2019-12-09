<template>
  <div class="container">
		<h3>Address Distance Calculator</h3>
		<div class="row">
			<div class="col-12">
				<p>This is a small Vue.js app that utilizes the Google Maps API to calculate the distance "as the crow flies" between two addresses.</p>
			</div>
		</div>

		<div class="row">
			<div class="col-12 col-md-6">
				<!-- Start Location -->
				<place-input
					v-model="startLocation"
					id="start"
					label="Start Location"
					ref="startLocation"
				/>
			</div>
		</div>
		<div class="row">
			<div class="col-12 col-md-6">
				<!-- End Location -->
				<place-input
					v-model="endLocation"
					id="end"
					label="Destination Location"
					ref="endLocation"
				/>
			</div>
		</div>
		
		<div class="row">
			<div class="col-4">
				<button 
					class="btn btn-primary"
					type="button"
					@click="getDistance"
				>
					Caclulate Distance
				</button>
			</div>
		</div>
  </div>
</template>

<script>
import 'bootstrap/dist/css/bootstrap.css';
import 'bootstrap-vue/dist/bootstrap-vue.css';

import PlaceInput from '~/components/PlaceInput.vue'

export default {
  components: {
    PlaceInput
	},

  data: {
    startLocation: '',
    endLocation: ''
	},

  methods: {
    /**
     * Calculates the distance between the start address and end address
     * @return {void}
     */
    async getDistance(){
      // Before we do anything, make sure the entered addresses are valid
      if(!this.validate()){
        return false;
      }
     
      // Get a google geocoder instance
			   const geocoder = new google.maps.Geocoder();
      
      // Make requests to geocode both addresses
      const startRequest = this.geocodeAddress(this.startLocation, geocoder);
      const endRequest = this.geocodeAddress(this.endLocation, geocoder);
      
      // Wait for both requests to complete before continuing
			   const geocodeResults = await Promise.all([startRequest, endRequest]);
      
      /**
       * Get the results of each request 
       * Successful requests will have resolved down to an object describing the latitude and longitude of the location
       * Unsuccessful requests will have resolved to null
       */
      const startLatLng = geocodeResults[0];
      const endLatLng = geocodeResults[1];
      
      // Make sure both addresses were geocoded successfully
      if(startLatLng === null || endLatLng === null){
        alert('Whoops, looks like something unexpected happened. Please try again later');
        return;
      }
      
      // Calculate the distance in meters using Google's geometry library -- note that this distance is "as the crow flies"
      let distance = google.maps.geometry.spherical.computeDistanceBetween(startLatLng, endLatLng);
      
      // Convert the distance to KM for convenience and display it
      distance = (distance / 1000).toFixed(3);
      alert(`Distance from start location to end location: ${distance}km`);
    },

   
    /**
     * Geocodes an address string
     *
     * @param {String} address - String representation of the address to geocode
     * @param {Google/Maps/Geocoder} geocoder - Google Maps Geocoder
     *
     * @return {Promise} Returns a promise that resolves when the geocoder request is complete
     */
    geocodeAddress(address, geocoder){
      return new Promise((resolve) => {
        // Define the request (limit the region to just the US)
        const request = {
          address: address,
          region: 'US'
        };

        // Geocode the address
        geocoder.geocode(request, (results, status) => {
          if(status === google.maps.GeocoderStatus.OK){
            // Geocoding was successful so resolve the promise with the LatLng object
            resolve(results[0].geometry.location);
          }else{
            // Geocoding was unsuccessful, so just resolve the promise with null
            resolve(null);
          }
        });
      });
    },
    
    /**
     * Confirms that both addresses are sufficiently valid for geocoding
     * @return {Boolean} True when addresses are valid, false otherwise
     */
    validate(){
      // Grab the start and end location components
      const start = this.$refs.startLocation;
      const end = this.$refs.endLocation;
      
      // Validate the start location component
      if(!start.validate()){
        alert('The starting location you entered is not valid');
        return false;
      }

      // Validate the end location component
      if(!end.validate()){
        alert('The destination location you entered is not valid');
        return false;
      }
      
      // Return true when both components are validated successfully
      return true;
    }
  }
}
</script>

<style>

</style>
