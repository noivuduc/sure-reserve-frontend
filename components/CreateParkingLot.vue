<template>
  <v-layout>
    <v-dialog v-model="dialog" @input="$emit('input')" persistent max-width="400">
      <v-card>
        <v-card-title class="headline">Create Parking Lot</v-card-title>
        <v-card-text>
          <v-combobox
            v-model="selectedCarPark"
            :items="carParks"
            label="Select Car park"
            height="45px"
            item-text="name"
            item-value="_id"
            outlined
            dense
          ></v-combobox>
          <v-text-field label="Parking Lot Name" placeholder="A" outlined v-model="parkingLotName"></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green darken-1" text @click="dialog = false">Cancel</v-btn>
          <v-btn color="green darken-1" text @click="create()">Create</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-layout>
</template>
<script>
export default {
  name: "CreateParkingLot",
  props: {
    carParks: Array,
    dialog: Boolean
  },
  data() {
    return {
      parkingLotName: null,
      selectedCarPark: null
    };
  },
  methods: {
    async create() {
      const data = {
        name: this.parkingLotName,
        code: this.parkingLotName,
        carParkId: this.selectedCarPark
      };
      const resp = await this.$axios.post("/parking-lot", data);
      if (resp.status == 201) {
        this.$swal({
          icon: "success",
          text: "Created!"
        });
      } else {
          this.$swal({
          icon: "error",
          text: "Error, Cannot create parking lot!"
        });
      }
      this.$emit("input");
    }
  }
};
</script>