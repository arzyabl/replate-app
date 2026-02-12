<script setup lang="ts">
import { fetchy } from "@/utils/fetchy";
import { onBeforeMount, ref } from "vue";

const props = defineProps(["username"]);

const loaded = ref(false);
const claims = ref<Array<Record<string, any>>>([]);
const isError = ref(false);

// Map of listingId to listing name
const listingNames = ref<Record<string, string>>({});

async function getListingName(listingId: string) {
  if (listingNames.value[listingId]) return listingNames.value[listingId];
  try {
    const listing = await fetchy(`/api/listings/${listingId}`, "GET");
    listingNames.value[listingId] = listing.name || listing.listing?.name || listingId;
    return listingNames.value[listingId];
  } catch {
    listingNames.value[listingId] = "Unknown";
    return "Unknown";
  }
}

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
  // Fetch listing names in parallel
  await Promise.all(
    claims.value.map(async (claim) => {
      const id = claim.item || claim.listingId;
      if (id) await getListingName(id);
    }),
  );
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
      <p>
        <strong>Listing: </strong>
        <span v-if="listingNames[claim.item || claim.listingId]">
          {{ listingNames[claim.item || claim.listingId] }}
        </span>
        <span v-else>(Loading...)</span>
      </p>
      <p><strong>Quantity:</strong> {{ claim.quantity }}</p>
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
