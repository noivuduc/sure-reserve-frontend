<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-btn color="primary" @click="dialog=true">Add Parking Lot</v-btn>
        <v-btn color="primary" @click="clearDatabase()">Clear Database</v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <p class="title"> Reservations</p>
        <v-data-table
          :headers="headers"
          :items="reservations"
          :items-per-page="15"
          class="elevation-1"
        ></v-data-table>
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
      headers: [
        { text: "Start Time", value: "startTime" },
        { text: "End Time", value: "endTime" },
        { text: "Car Park", value: "carPark" },
        { text: "Parking Lot", value: "parkingLot" }
      ],
      selectedCarPark: null,
      dialog: false,
      reservations: [],
      parkingLotName: null
    };
  },
  mounted() {
    this.getAllCarParks();
    this.getAllReservations();
  },
  methods: {
    async getAllCarParks() {
      const resp = await this.$axios.get("/car-park");
      if (resp.status == 200) {
        this.carParks = resp.data;
      }
    },
    async getAllReservations() {
      const resp = await this.$axios.get("/reservation");
      if (resp.status == 200) {
        const data = resp.data;
        const reservations = !data
          ? []
          : data.map(element => this.beautifyReservation(element));
        this.reservations = reservations;
      }
    },

    async clearDatabase() {
      const resp = await this.$axios.delete("/reservation");
      this.$swal({
        icon: "success",
        text: "Database is cleared"
      });
      this.getAllReservations()
    },

    beautifyReservation(reservation) {
      const parkingLot = reservation.parkingLotId;
      const carPark = parkingLot.carParkId;
      return {
        startTime: reservation.start_time,
        endTime: reservation.end_time,
        parkingLot: parkingLot.name,
        carPark: carPark.name
      };
    }
  }
};
</script>