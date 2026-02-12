<script setup lang="ts">
import { fetchy } from "@/utils/fetchy";
import { onBeforeMount, ref } from "vue";

const props = defineProps(["username"]);

const loaded = ref(false);
const claims = ref<Array<Record<string, any>>>([]);
const isError = ref(false);

async function getClaims() {
  let results;
  try {
    results = await fetchy("/api/claims", "GET", { query: { claimer: props.username } });
  } catch (error) {
    console.error("Failed to fetch claims:", error);
    isError.value = true;
    return;
  }
  claims.value = results;
}

onBeforeMount(async () => {
  await getClaims();
  loaded.value = true;
});
</script>

<template>
  <p>Here you can view the claims you have made.</p>
  <section class="claims-container" v-if="loaded && claims.length !== 0">
    <article v-for="claim in claims" :key="claim._id" class="claim-item">
      <p><strong>Listing ID:</strong> {{ claim.listingId }}</p>
      <p><strong>Quantity:</strong> {{ claim.quantity }}</p>
      <!-- Add more claim details as needed -->
    </article>
  </section>
  <p v-else-if="loaded && !isError">No claims yet.</p>
  <p v-else-if="isError">Failed to load claims. Please try again later.</p>
  <p v-else>Loading...</p>
</template>

<style scoped>
.claims-container {
  display: flex;
  flex-direction: column;
  padding: 1em;
}

.claim-item {
  border: 1px solid #ccc;
  padding: 1em;
  border-radius: 5px;
  margin-bottom: 1em;
}
</style>
