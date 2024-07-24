<template>
  <div class="box">
    <div class="container">
      <img src="../assets/images/The_M_Scale_Mark_orange_rgb.svg">
    </div>
    <h1>{{ msg }}</h1>
    <!-- <div style="display: flex;justify-content: space-evenly;max-width: 600px;margin: 0 auto;"> -->
      <button type="button" @click="convertJSONtoCSV(data)" class="btn mt-5" style="background-color: #f49600;font-weight:600">Download Reviews</button>
      <!-- <button type="button" @click="downloadVenues" class="btn mt-5" style="background-color: #313ab8;color:#fff;font-weight:600">Download Venues</button> -->
    <!-- </div> -->
  </div>
</template>

<script>
import { initializeApp } from "firebase/app";
import { getDatabase, ref, onValue } from "firebase/database";
import * as XLSX from "xlsx" // Import the xlsx library

export default {
  name: 'LandingPage',
  props: {
    msg: String
  },

  data() {
    return {
      data: null,
      venuesData: null
    };
  },
  async created() {
    const response = await fetch('/.netlify/functions/firebaseConfig');
    const firebaseConfig = await response.json();
    // Initialize Firebase
    const app = initializeApp(firebaseConfig);
    // Initialize Realtime Database and get a reference to the service
    const db = getDatabase(app);
    const venues = ref(db, 'venues');
    onValue(venues, (snapshot) => {
      this.data = snapshot.val();
    });
  },
  methods: {
    formatDate: function (timestamp) {
      const dateObj = new Date(timestamp);
      const year = dateObj.getFullYear();
      const month = String(dateObj.getMonth() + 1).padStart(2, '0'); // Month is 0-indexed, so add 1
      const day = String(dateObj.getDate()).padStart(2, '0');
      const hours = String(dateObj.getHours()).padStart(2, '0');
      const minutes = String(dateObj.getMinutes()).padStart(2, '0');

      return `${year}-${month}-${day} ${hours}:${minutes}`;
    },
    convertJSONtoCSV: function (jsonData) {
      const transformedData = [];
      const data = Object.values(jsonData);
      data.forEach(location => {
        const locationName = location.place;
        const reviews = Object.values(location.reviews);

        reviews.forEach(review => {
          transformedData.push({
            Date: this.formatDate(review.timestamp),
            'User email': review.userId,
            Location: locationName,
            'Review Rating': review.vote,
            Comments: review.description
          });
        });
      });

      const worksheet = XLSX.utils.json_to_sheet(transformedData);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Sheet1');

      // Generate buffer with the workbook in XLSX format
      const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' });

      // Convert buffer to Blob
      const blob = new Blob([excelBuffer], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });

      const url = URL.createObjectURL(blob);

      const downloadLink = document.createElement('a');
      downloadLink.href = url;
      downloadLink.download = 'db_dump.xlsx'; // Set the desired file name with .xlsx extension
      downloadLink.click();

      // Release the object URL after the download link is clicked
      URL.revokeObjectURL(url);
    },
    async fetchVenueDetails(placeId) {
      const url = `/.netlify/functions/getVenueDetails?placeId=${placeId}`;
      const response = await fetch(url);
      const data = await response.json();
      return data.result;
    },
    async downloadVenues() {
      const transformedData = [];
      const venueEntries = Object.entries(this.data);
      
      for (const [venue] of venueEntries) {
        const placeId = venue.placeId;
        const details = await this.fetchVenueDetails(placeId);
        if (details) {
          transformedData.push({
            'Venue Name': details.name,
            'Address': details.formatted_address
          });
        }
      }

      const worksheet = XLSX.utils.json_to_sheet(transformedData);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Venues');

      // Generate buffer with the workbook in XLSX format
      const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' });

      // Convert buffer to Blob
      const blob = new Blob([excelBuffer], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });

      const url = URL.createObjectURL(blob);

      const downloadLink = document.createElement('a');
      downloadLink.href = url;
      downloadLink.download = 'venues.xlsx'; // Set the desired file name with .xlsx extension
      downloadLink.click();

      // Release the object URL after the download link is clicked
      URL.revokeObjectURL(url);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
