<template>
  <form class="flex justify-center mt-20">
    <div class="flex flex-wrap -mx-3">
      <div class="flex items-center border-b border-white-500 py-2 mr-10">
        <select
          v-model="search.addressType"
          class="bg-transparent text-white focus:outline-none"
        >
          <option
            value=""
            class="text-white bg-gray-800 rounded"
            v-for="option in addressTypes"
            :key="option"
          >
            {{ option }}
          </option>
        </select>
      </div>

      <div class="flex items-center border-b border-white-500 py-2">
        <i class="fa fa-question-circle text-white mr-2 cursor-pointer"></i>
        <input
          ref="searchInput"
          class="appearance-none bg-transparent border-none w-full text-white mr-3 py-1 px-2 leading-tight focus:outline-none"
          type="text"
          v-model="search.searchValue"
          placeholder="Address"
        />
        <i
          class="fa fa-times-circle text-white mr-2 cursor-pointer"
          @click="cleanSearchInput()"
          v-if="search.searchValue != ''"
        ></i>
        <i
          class="fa fa-plus text-white mr-2 cursor-pointer"
          @click="additionalInput = true"
          v-if="additionalInputIcon"
        ></i>
      </div>

      <div class="flex items-center border-b border-white-500 py-2 ml-10">
        <input
          class="appearance-none bg-transparent border-none text-white mr-3 py-1 px-2 leading-tight focus:outline-none"
          type="text"
          v-model="addressNo"
          placeholder="N°"
          v-if="addressNoInput"
          v-on:blur="addressNumberLosesFocus()"
        />

        <input
          class="appearance-none bg-transparent border-none text-white mr-3 py-1 px-2 leading-tight focus:outline-none"
          type="text"
          v-model="additionalValue"
          placeholder="Add another value"
          v-if="additionalInput"
          v-on:blur="adittionalInputLosesFocus()"
        />
      </div>
    </div>
  </form>
</template>

<script>
const mapAPI = process.env.mapAPI; // from nuxt.config.js  env:{}
export default {
  head() {
    return {
      script: [
        {
          src: `https://maps.googleapis.com/maps/api/js?key=${mapAPI}&libraries=places`,
        },
      ],
    };
  },

  data() {
    return {
      additionalInput: false,
      additionalInputIcon: false,
      addressNoInput: false,
      addressNo: "",
      additionalValue: "",
      search: {
        searchValue: "",
        addressType: "",
      },
      addressTypes: ["Residential", "Domicile", "Legal", "Operational"],
    };
  },

  mounted() {
    let searchInput = this.$refs.searchInput;
    const autocomplete = new google.maps.places.Autocomplete(searchInput); // Autocomplete Object

    // Listen when user select a suggetion from the locations list
    autocomplete.addListener("place_changed", () => {
      this.addressNoInput = true; // Add the input to enter the Address Number
      this.search.searchValue = autocomplete.getPlace().formatted_address; // Set the place select to the Address Input
    });

    searchInput.addEventListener("input", () => {
      // When the Address input is empty remove Address Number input, + icon and clean the address number input
      if (this.search.searchValue === "") {
        this.addressNoInput = false;
        this.additionalInputIcon = false;
        this.addressNo = "";
      }
    });
  },

  methods: {
    cleanSearchInput() {
      this.search.searchValue = "";
      this.addressNo = "";
      this.additionalValue = "";
      this.addressNoInput = false;
      this.additionalInputIcon = false;
      this.additionalInput = false;
    },
    // When the Address number loses the focus its value append at the start of address input value
    addressNumberLosesFocus() {
      let addressNum = this.addressNo;
      // Check if the Address Input is not empty
      if (addressNum != "") {
        this.search.searchValue = addressNum + ", " + this.search.searchValue;
        this.addressNo = ""; // Clean the Address number input
        this.addressNoInput = false; // Remove Address number input
        this.additionalInputIcon = true; // add + icon
      }
    },

    // When the Additional value input loses the focus its value append at the end of address input value
    adittionalInputLosesFocus() {
      let searchInput = this.$refs.searchInput;
      let additionalValue = this.additionalValue;
      if (additionalValue != "") {
        this.search.searchValue =
          this.search.searchValue + ", " + additionalValue;
        this.additionalInput = false;
        this.additionalValue = "";
        searchInput.focus();
      }
    },
  },
};
</script>

<style>
.pac-container {
  background-color: rgb(55, 55, 55);
}

.pac-item-query {
  color: white;
}

.pac-item {
  border-top: none;
}

.pac-logo:after {
  background-image: none;
}

.pac-item:hover {
  background-color: rgb(119, 119, 119);
}
</style>