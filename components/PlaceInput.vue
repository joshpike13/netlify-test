<template>
	<div class="input-wrapper">
		<label :for="id">{{ label }}</label>
		<div class="form-group row">

			<!-- Address Field -->
			<div class="col-8">
				<input
				v-model="address"
				:id="id"
				placeholder="Street Address"
				type="text" 
				class="form-control"
				@input="$emit('input', formattedAddress)"
				>
			</div>


			<!-- Zip Code Field -->
			<div class="col-4">
				<input
					v-model="zip"
					class="form-control"
					placeholder="Zip Code"
					type="text"
					@input="$emit('input', formattedAddress)"
				>
			</div>
		</div>
	</div>
</template>

<script>
export default {
  props: {
		  // A unique ID to distinguish this input from the others
    id: {
			   required: true,
			   type: String
    },
		  // A label to identify this input
		  label: {
			   required: true,
			   type: String
		  },
		  // Model value from parent component
    value: {
      required: true,
      type: String
    }
	},

  data(){
    return {
      autocomplete: null,
      address: '',
      zip: ''
    }
	},

  computed: {

    /**
     * Composes the address and zip code into a formatted string
     * This value is emitted back up to the parent any time an input event occurs
     *
     * @return {String} A nicely formatted address string
     */
    formattedAddress(){
      // Compose the address and zip into a nicely formatted string
      return `${this.address}, ${this.zip}`;
    }
	},

  /**
   * Executed when this component is mounted on the DOM
   * Initializes address lookup on the input element
   * 
   * @return {void}
   */
  mounted(){
    // Get the input element using the ID prop and make sure it exists
    const inputElement = document.getElementById(this.id);
    if(inputElement){

      // Define some configuration settings to limit autocomplete results down to addresses in the US
      const autocompleteConfiguration = {
        componentRestrictions: {
          country: 'us'
        },
        types: ['address']
      };
     
      // Attach the Google autocomplete to the input
     	this.autocomplete = new google.maps.places.Autocomplete(inputElement, autocompleteConfiguration);

     	// Limit the fields returned from the API to just address components
			   this.autocomplete.setFields(['address_components']);
     
      // When an address is selected, fire the local callback method so we can process further
			   this.autocomplete.addListener('place_changed', this.placeChanged);
    }
	},

  methods: {
    /**
     * Executed when an address is selected from the API lookup and formats the results
     * @return {void}
     */
    placeChanged(){
      // Get the Google Place and grab the returned address components
      const place = this.autocomplete.getPlace();
      const items = place.address_components;

      // Define some variables to pull out the necessary data
      let number = '';
      let street = '';
      let zip = '';
     
      // Iterate through the address components and grab street address and zip code
      items.forEach((item) => {
        const type = item.types[0];
        switch(type){
          case 'route':
            street = item.short_name;
            break;
          case 'street_number':
            number = item.short_name;
            break;
          case 'postal_code':
            zip = item.short_name;
            break;
          default:
            break;
        }
      });
      
      // Set the values in the component's local data properties
      this.address = `${number} ${street}`;
      this.zip = zip;
      
      // Emit an input event back up to the parent so it stays in sync
      this.$emit('input', this.formattedAddress);
    },
 
    /**
     * Checks whether the data in this component is sufficiently valid for geocoding
     * @return {Boolean} True when component data is valid, false otherwise
     */
    validate(){
      // Rudimentary address check - make sure at least 5 characters were entered
      if(this.address.length < 5){
        return false;
      }
     
      // Zip code - make we got exactly 5 digits
      const regex = /^([0-9]{5})$/;
      if(this.zip.length !== 5 || !regex.test(this.zip)){
        return false;
      }
      
      // If we made it here then validation was successful
      return true;
    }
  }
}
</script>

<style>

</style>