<template>
  <v-container fluid>
    <v-stepper v-model="e6" vertical>
      <v-stepper-step :complete="e6 > 1" step="1" editable>
        Your checkin time:
        <b>{{ startTime }}</b>
      </v-stepper-step>

      <v-stepper-content step="1">
        <v-card class="mb-5" elevation="0">
          <v-date-picker
            v-model="date"
            full-width
            :landscape="$vuetify.breakpoint.smAndUp"
            class="mt-4"
          ></v-date-picker>
          <v-combobox
            class="mt-10"
            height="45"
            v-model="time"
            :items="times"
            label="Check in time"
            outlined
            dense
          ></v-combobox>
          <p>Your selection time: {{ startTime }}</p>
        </v-card>
        <v-btn color="primary" @click="e6 = 2">Continue</v-btn>
      </v-stepper-content>

      <v-stepper-step :complete="e6 > 2" step="2" editable>Select Car Park Area</v-stepper-step>

      <v-stepper-content step="2">
        <v-card class="mb-12" elevation="0">
          <v-row>
            <v-col v-for="(carPark, index) in carParks" :key="index" cols="12" md="4" lg="4" sm="12">
              <v-hover v-slot:default="{ hover }">
                <v-card
                  :color="selectedCarPark && selectedCarPark._id === carPark._id ? '#00b0ff' : '#1F7087'"
                  dark
                  :elevation="hover ? 12 : 2"
                  @click="selectCarPark(carPark)"
                >
                  <div class="d-flex flex-no-wrap justify-space-between">
                    <div>
                      <v-card-title class="headline">{{ carPark.name }}</v-card-title>
                    </div>
                    <v-avatar class="ma-3" size="125" tile>
                      <v-img :src="carPark.image"></v-img>
                    </v-avatar>
                  </div>
                </v-card>
              </v-hover>
            </v-col>
          </v-row>
        </v-card>
        <v-btn color="primary" @click="e6 = 3">Continue</v-btn>
      </v-stepper-content>

      <v-stepper-step :complete="e6 > 3" step="3" editable>Confirm Reservation</v-stepper-step>

      <v-stepper-content step="3">
        <v-card class="mb-12" elevation="0" v-if="selectedCarPark && !recommended">
          <v-row align="center" justify="center">
            <v-col cols="12" md="8" lg="6" sm="12">
              <v-card class="mx-auto my-12" max-width="450">
                <v-img height="250" :src="selectedCarPark.image"></v-img>

                <v-card-title>{{ selectedCarPark.name }}</v-card-title>

                <v-divider class="mx-4"></v-divider>

                <v-card-title>Reservation details</v-card-title>

                <v-card-text>
                  <p class="subtitle-1 ma-0">
                    Check in:
                    <span class="success--text">{{ startTime }}</span>
                  </p>
                  <p class="subtitle-1 ma-0">
                    Check out:
                    <span class="success--text">{{ endTime }}</span>
                  </p>
                </v-card-text>

                <v-card-actions>
                  <v-btn color="deep-purple lighten-2" text @click="reserve(reserveData)">Reserve</v-btn>
                </v-card-actions>
              </v-card>
            </v-col>
          </v-row>
        </v-card>
        <v-card class="mb-12" elevation="0" v-if="recommended && recommended.length > 0">
          <v-card-title>Suggestion parking slot for requested time: {{ startTime }}</v-card-title>
          <v-row align="center" justify="center">
            <v-col cols="12" md="4" lg="4" sm="12" v-for="(recommend, index) in recommended" :key="index">
              <v-card class="mx-auto my-12" max-width="450">
                <v-img height="250" :src="recommend.foundCarPark.image"></v-img>

                <v-card-title>{{ recommend.foundCarPark.name }}</v-card-title>

                <v-divider class="mx-4"></v-divider>

                <v-card-title>Reservation details</v-card-title>

                <v-card-text>
                  <p class="subtitle-1 ma-0">
                    Check in:
                    <span class="success--text">{{ recommend.start_time }}</span>
                  </p>
                  <p class="subtitle-1 ma-0">
                    Check out:
                    <span class="success--text">{{ recommend.end_time }}</span>
                  </p>
                </v-card-text>

                <v-card-actions>
                  <v-btn
                    color="deep-purple lighten-2"
                    text
                    @click="reserve({start_time: recommend.start_time, carParkId: recommend.foundCarPark._id})"
                  >Reserve</v-btn>
                </v-card-actions>
              </v-card>
            </v-col>
          </v-row>
        </v-card>
      </v-stepper-content>
    </v-stepper>
    <loading
      :active.sync="isLoading"
      :can-cancel="false"
      :is-full-page="true"
      :background-color="'#FFAB40'"
      :color="'#fff'"
      :loader="'dots'"
      :opacity="0.8"
    ></loading>
  </v-container>
</template>

<script>
import * as moment from "moment";
import Loading from "vue-loading-overlay";
export default {
  components: {
    Loading
  },
  props: {
    source: String
  },
  data() {
    return {
      date: new Date().toISOString().substr(0, 10),
      carParks: [],
      e6: 1,
      isLoading: false,
      selectedCarPark: null,
      time: "12:00",
      recommended: null
    };
  },
  computed: {
    times() {
      let time = [];
      let startTime = moment()
        .minutes(0)
        .hours(0);
      while (
        startTime.isSameOrBefore(
          moment()
            .hours(23)
            .minutes(30)
        )
      ) {
        time.push(startTime.format("HH:mm"));
        startTime.add(15, "minutes");
      }
      return time;
    },
    startTime() {
      return `${this.date} ${this.time}`;
    },
    endTime() {
      return moment(this.startTime)
        .add(3, "hours")
        .format("YYYY-MM-DD HH:mm");
    },
    reserveData() {
      return {
        start_time: this.startTime,
        carParkId: this.selectedCarPark._id
      };
    }
  },
  mounted() {
    this.getCarParks();
  },
  methods: {
    async getCarParks() {
      try {
        const resp = await this.$axios.get("/car-park");
        this.carParks = Object.assign([], resp.data);
      } catch (error) {
        console.log(error);
      }
    },
    async reserve(body) {
      try {
        this.isLoading = true;
        const resp = await this.$axios.post("/reservation", body);
        this.isLoading = false;
        if (resp.status == 201) {
          const data = resp.data;
          if (data.booked) {
            this.recommended = null;
            this.selectedCarPark = null;
            this.e6 = 1;
            this.$swal({
              text: "Successfully",
              icon: "success"
            });
            this.$router.push({path: '/reservations/' + data.reservationId})
          } else {
            this.$swal({
              text:
                "This car park is fully booked. Please see some available suggested below",
              icon: "warning"
            });
            this.recommended = data.recommended;
          }
        } else {
          this.$swal({
            text: "Error",
            icon: "error"
          });
        }
      } catch (error) {
        this.isLoading = false;
        console.log(error);
        this.$swal({
          text: error.response.data.message,
          icon: "error"
        });
      }
    },
    selectCarPark(carPark) {
      this.selectedCarPark = Object.assign({}, carPark);
    }
  }
};
</script>
<style scoped>
.car-selected {
  border-color: chocolate !important;
}
</style>