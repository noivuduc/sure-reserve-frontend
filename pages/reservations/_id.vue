<template>
  <v-container>
    <v-row align="center" justify="center">
      <v-col cols="8">
        <p class="title success--text" align="center">Reservation confirmed</p>
        <v-card class="mx-auto my-12" max-width="450" v-if="loaded">
          <v-img height="250" :src="carPark.image"></v-img>

          <v-card-title>{{ carPark.name }}</v-card-title>
          <v-card-subtitle>Lot: {{ parkingLot.name }}</v-card-subtitle>

          <v-divider class="mx-4"></v-divider>

          <v-card-title>Reservation details</v-card-title>

          <v-card-text>
            <p class="subtitle-1 ma-0">
              Check in:
              <span class="success--text">{{ reservation.start_time }}</span>
            </p>
            <p class="subtitle-1 ma-0">
              Check out:
              <span class="success--text">{{ reservation.end_time }}</span>
            </p>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
export default {
  data() {
    return {
      carPark: null,
      parkingLot: null,
      reservation: null,
      loaded: false
    };
  },
  mounted() {
    const params = this.$route.params;
    console.log(params);
    const resultId = params.id;
    this.getReservation(resultId);
  },
  methods: {
    async getReservation(id) {
      const resp = await this.$axios.get("/reservation/" + id);
      if (resp.status == 200) {
        this.loaded = true;
        const data = resp.data;
        this.reservation = data;
        this.parkingLot = data.parkingLotId;
        this.carPark = this.parkingLot.carParkId;
      }
    }
  }
};
</script>