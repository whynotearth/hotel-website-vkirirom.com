<template>
  <v-card class="d-flex flex-column dark light--text" tile :elevation="0">
    <div class="d-flex flex-column flex-grow-1">
      <div class="position-relative hero-dialog--hero">
        <v-img
          :aspect-ratio="376 / 192"
          :max-height="192"
          :max-width="'100%'"
          :src="transformCloudinaryUrl(resort.featuredImage, 'f_auto')"
        ></v-img>
        <div v-if="hasCancelButton" class="position-absolute mx-4 mt-4 hero-dialog--toolbar">
          <v-btn class="ma-0" x-small fab color="rgba(0,0,0,0.4)" depressed @click="$emit('booking-cancel')">
            <v-icon color="white">close</v-icon>
          </v-btn>
        </div>
        <div class="position-absolute hero-dialog--title text-center brand-2--text w-100">
          <h2 class="display-1 mb-0 font-weight-bold">
            {{ resort.title }}
          </h2>
        </div>
      </div>

      <v-card dark color="dark" tile :elevation="0" class="px-4 pt-6 flex-grow-1 d-flex">
        <div class="d-flex flex-column flex-grow-1 justify-space-between body-2 font-weight-light">
          <div>
            <!-- # guests -->
            <v-row no-gutters class="mb-6">
              <v-col cols="12">
                <h3 class="d-flex align-center title font-weight-medium mb-0 booking-confirm--h3-title">
                  <v-icon color="light" class="icon mr-4">person</v-icon>
                  <span class="mr-2">{{ guests }}</span
                  ><span>Guests</span>
                </h3>
              </v-col>
            </v-row>
            <!-- from date => to date -->
            <v-row no-gutters class="mb-8">
              <v-col>
                <div class="d-flex title font-weight-medium booking-confirm--h3-title">
                  <v-icon color="light" class="icon mr-4">event</v-icon>
                  <div class="d-flex align-center flex-grow-1">
                    <div class="confirm-booking--date">{{ formatDate(dateOne, 'ddd, D MMM') }}</div>
                    <v-icon color="light" class="icon mx-3">arrow_right_alt</v-icon>
                    <div class="confirm-booking--date">{{ formatDate(checkOut, 'ddd, D MMM') }}</div>
                  </div>
                </div>
              </v-col>
            </v-row>

            <v-divider class="light-border mb-6"></v-divider>

            <!-- # nights total -->
            <v-row no-gutters class="mb-4">
              <v-col
                ><p class="mb-0 title font-weight-medium booking-confirm--h3-title">
                  {{ prices.length }} nights total
                </p></v-col
              >
            </v-row>

            <!-- price list -->
            <div class="mb-6">
              <div class="mb-8">
                <v-row class="confirm-dates--price-row" no-gutters v-for="price in prices" v-bind:key="price.id">
                  <v-col xs6>{{ formatDate(price.date, 'ddd, D MMM') }}</v-col>
                  <v-col xs6 class="text-right ">${{ price.amount }}</v-col>
                </v-row>
              </div>
              <!-- VAT -->
              <v-row class="confirm-dates--price-row" no-gutters>
                <v-col xs6 class="title font-weight-medium booking-confirm--h3-title">VAT (10%)</v-col>
                <v-col xs6 class="text-right ">${{ computedVAT }}</v-col>
              </v-row>
            </div>
          </div>

          <!-- non-sticky bar -->
          <div class="section-2 submit-bar--non-sticky pb-9">
            <v-divider class="light-border mb-6"></v-divider>
            <!-- total -->
            <v-row no-gutters class="mt-6">
              <v-col xs6>
                <h3 class="title font-weight-semiblack brand-2--text mb-0">Total</h3>
              </v-col>
              <v-col xs6 class="text-right">
                <h3 class="title mb-0">
                  <span>${{ computedTotalPrice }}</span>
                </h3>
              </v-col>
            </v-row>

            <v-btn
              v-if="hasConfirmButton"
              @click="submit"
              x-large
              block
              color="primary"
              dark
              class="text-transform-none font-weight-bold dark--text mt-6"
            >
              <v-spacer></v-spacer>
              <span>Confirm Booking</span>
              <v-spacer></v-spacer>
              <v-icon>keyboard_arrow_right</v-icon>
            </v-btn>
          </div>
        </div>
      </v-card>
    </div>

    <!-- sticky bar -->
    <div class="submit-bar--sticky d-none" :class="{ 'pb-9': !hasConfirmButton }">
      <div class="px-4">
        <v-divider class="light-border mb-6 mt-1"></v-divider>
        <!-- total -->
        <v-row no-gutters>
          <v-col xs6>
            <h3 class="title mb-0">Total</h3>
          </v-col>
          <v-col xs6 class="text-right">
            <h3 class="title mb-0">
              <span>${{ computedTotalPrice }}</span>
            </h3>
          </v-col>
        </v-row>
      </div>

      <v-btn
        v-if="hasConfirmButton"
        @click="submit"
        x-large
        block
        color="primary"
        dark
        class="text-transform-none font-weight-bold dark--text mt-6"
      >
        <v-spacer></v-spacer>
        <span>Confirm Booking</span>
        <v-spacer></v-spacer>
        <v-icon>keyboard_arrow_right</v-icon>
      </v-btn>
    </div>
  </v-card>
</template>

<script lang="ts">
import Vue from 'vue';
import store from '@/store';
import { formatDate } from '@/helpers';

export default Vue.extend({
  name: 'booking-confirm-dates',
  props: {
    hasConfirmButton: {
      type: Boolean,
      default: false
    },
    hasCancelButton: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    resort() {
      return (this as any).$store.getters['booking/bookingInfo'].resort;
    },
    dateOne() {
      return (this as any).$store.getters['booking/bookingInfo'].dateOne;
    },
    dateTwo() {
      return (this as any).$store.getters['booking/bookingInfo'].dateTwo;
    },
    checkOut() {
      return (this as any).$store.getters['booking/bookingInfo'].checkOut;
    },
    prices() {
      return (this as any).$store.getters['booking/prices']({ decimalDigits: 0 });
    },
    computedVAT() {
      return (this as any).$store.getters['booking/computedVAT']({ decimalDigits: 0 });
    },
    computedTotalPrice() {
      const options = {
        hasVAT: true,
        decimalDigits: 0
      };
      return (this as any).$store.getters['booking/computedTotalPrice'](options);
    },
    guests() {
      return (this as any).$store.getters['booking/bookingInfo'].guests.total;
    }
  },
  methods: {
    formatDate,
    submit() {
      this.$emit('booking-close');
      this.$router.push({ name: 'booking-review-rules' });
    }
  }
});
</script>

<style lang="scss">
@import '@/styles/utility.scss';
</style>
<style lang="scss" scoped>
@import '@/styles/dialog-with-hero.scss';
@import '@/styles/sticky-submit-bar.scss';
.booking-confirm--h3-title {
  line-height: 1.25;
}
</style>
