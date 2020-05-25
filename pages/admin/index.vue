<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-btn color="primary" @click="dialog=true">Add Parking Lot</v-btn>
        <v-btn color="primary" @click="clearDatabase()">Clear Database</v-btn>
      </v-col>
    </v-row>
    <create-parking-lot :carParks="carParks" v-model="dialog" :dialog="dialog"></create-parking-lot>
  </v-container>
</template>
<script>
import CreateParkingLot from "@/components/CreateParkingLot";
export default {
  components: {
    CreateParkingLot
  },
  data() {
    return {
      carParks: [],
      selectedCarPark: null,
      dialog: false,
      parkingLotName: null
    };
  },
  mounted() {
    this.getAllCarParks();
  },
  methods: {
    async getAllCarParks() {
      const resp = await this.$axios.get("/car-park");
      if (resp.status == 200) {
        this.carParks = resp.data;
      }
    },

    async clearDatabase() {
      const resp = await this.$axios.delete("/reservation");
      this.$swal({
        icon: "success",
        text: "Database is cleared"
      });
    }
  }
};
</script>