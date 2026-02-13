<script setup lang="ts">
import ListingListComponent from "@/components/Listing/ListingListComponent.vue";
import UserOfferListComponent from "@/components/Offer/UserOfferListComponent.vue";
import UserProfileComponent from "@/components/Profile/UserProfileComponent.vue";
import RequestListComponent from "@/components/Request/RequestListComponent.vue";
import { useUserStore } from "@/stores/user";
import { storeToRefs } from "pinia";
import { computed, ref } from "vue";
import { useRoute } from "vue-router";
import UserClaimsByUserComponent from "../components/Claiming/UserClaimsByUserComponent.vue";

const activeSection = ref("listings"); // Default to "listings"
const { currentUsername } = storeToRefs(useUserStore());
const route = useRoute();
const profileUsername = Array.isArray(route.params.id) ? route.params.id[0] : route.params.id;
const isOwnProfile = computed(() => currentUsername.value === profileUsername);
</script>
<template>
  <main>
    <UserProfileComponent :userId="route.params.id" />

    <!-- Clickable Headers for Sections -->
    <div class="tabs">
      <h1 :class="{ active: activeSection === 'listings' }" @click="activeSection = 'listings'">Listings</h1>
      <h1 :class="{ active: activeSection === 'requests' }" @click="activeSection = 'requests'">Requests</h1>
      <template v-if="isOwnProfile">
        <h1 :class="{ active: activeSection === 'offers' }" @click="activeSection = 'offers'">Offers</h1>
        <h1 :class="{ active: activeSection === 'claims' }" @click="activeSection = 'claims'">Claims</h1>
      </template>
    </div>

    <!-- Conditional Rendering Based on Active Section -->
    <section v-if="activeSection === 'listings'">
      <p>Available listings</p>
      <ListingListComponent :username="profileUsername" />
      <p>Historic listings</p>
      <ListingListComponent :username="profileUsername" :historic="true" />
    </section>
    <section v-else-if="activeSection === 'requests'">
      <p>Ongoing requests</p>
      <RequestListComponent :username="profileUsername" />
      <p>Historic requests</p>
      <RequestListComponent :username="profileUsername" :historic="true" />
    </section>
    <section v-else-if="activeSection === 'offers' && isOwnProfile">
      <UserOfferListComponent :username="profileUsername" />
    </section>
    <section v-else-if="activeSection === 'claims' && isOwnProfile">
      <UserClaimsByUserComponent :username="profileUsername" />
    </section>
  </main>
</template>
<style scoped>
main {
  display: flex;
  flex-direction: column;
  align-items: center;
}

h1 {
  text-align: center;
}

.tabs {
  display: flex;
  gap: 2rem; /* Spacing between the headers */
  margin-bottom: 0px;
}

h1.active {
  border-bottom: 2px solid;
}

p {
  padding-top: 2em;
  font-size: 1.2em;
}

.thumb-container {
  width: 1000px;
  display: grid;
  gap: 5px;
  padding: 10px;
  align-items: center;
  margin: 0px;
  grid-template-columns: repeat(5, 1fr); /* Five equal columns */
}
</style>
