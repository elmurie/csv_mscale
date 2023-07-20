<template>
  <div class="box">
    <div class="container">
      <img
        src="https://themscale.com/wp-content/uploads/2023/05/ef12556d-4e7c-4d80-9241-dcf8aa1b720a-removebg-preview.png">
    </div>
    <h1>{{ msg }}</h1>
    <button type="button" @click="convertJSONtoCSV(data)" class="btn btn-danger mt-5">Download</button>
  </div>
</template>

<script>
import { initializeApp } from "firebase/app";
import { getDatabase, ref, onValue } from "firebase/database";
import * as XLSX from "xlsx" // Import the xlsx library
const firebaseConfig = {
  apiKey: process.env.VUE_APP_API_KEY,
  authDomain: process.env.VUE_APP_AUTH_DOMAIN,
  databaseURL: process.env.VUE_APP_DATABASE_URL,
  projectId: process.env.VUE_APP_PROJECT_ID,
  storageBucket: process.env.VUE_APP_STORAGE_BUCKET,
  messagingSenderId: process.env.VUE_APP_MESSAGING_SENDER_ID,
  appId: process.env.VUE_APP_APP_ID,
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);


export default {
  name: 'LandingPage',
  props: {
    msg: String
  },

  data() {
    return {
      data: null
    };
  },
  created() {
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
      downloadLink.download = 'output.xlsx'; // Set the desired file name with .xlsx extension
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
