<script setup lang="ts">
import { useUserStore } from "@/stores/user";
import { fetchy } from "@/utils/fetchy";
import { storeToRefs } from "pinia";
import { computed, onBeforeMount, ref, watch } from "vue";
import ListingThumbComponent from "./ListingThumbComponent.vue";

const props = defineProps(["searchTerm", "username", "historic", "tags"]);

const { isLoggedIn } = storeToRefs(useUserStore());
const loaded = ref(false);
const tagLoading = ref(false);
let listings = ref<Array<Record<string, string>>>([]);
const itemsWithTag = ref<Array<string>>();

async function getListings() {
  let results;
  try {
    if (props.username) {
      results = await fetchy("/api/listings", "GET", { query: { author: props.username } });
    } else {
      results = await fetchy("/api/listings", "GET");
    }
  } catch (_) {
    return;
  }
  listings.value = results;
}

async function getTagged() {
  tagLoading.value = true;
  try {
    if (props.tags && props.tags.length > 0) {
      const tagResults = await Promise.all(
        props.tags.map((tag) => {
          const result = fetchy(`/api/tagged/${tag}`, "GET");
          result.then((res) =>
            console.log(
              `Fetched for tag '${tag}':`,
              res.map((t: any) => t.toString()),
            ),
          );
          return result;
        }),
      );
      const sets = tagResults.map((arr, idx) => {
        const set = new Set(arr.map((t: any) => t.toString()));
        console.log(`Set for tag '${props.tags[idx]}':`, Array.from(set));
        return set;
      });
      if (sets.length === 1) {
        itemsWithTag.value = Array.from(sets[0]);
        console.log("Single tag, items:", itemsWithTag.value);
      } else if (sets.length > 1) {
        let intersection = sets[0];
        for (let i = 1; i < sets.length; i++) {
          intersection = new Set(Array.from(intersection).filter((x) => sets[i].has(x)));
        }
        itemsWithTag.value = Array.from(intersection);
        console.log("Intersection of tags, items:", itemsWithTag.value);
      } else {
        itemsWithTag.value = [];
        console.log("No tags selected, items: []");
      }
    } else {
      itemsWithTag.value = undefined;
    }
  } catch {
    itemsWithTag.value = [];
  } finally {
    tagLoading.value = false;
  }
}

watch(
  // Watch for changes in the "tags" prop
  () => props.tags,
  async (newTags, oldTags) => {
    await getTagged();
  },
  { immediate: true }, // Fetch immediately when the component is mounted
);

const filteredListings = computed(() => {
  const searchTerm = props.searchTerm?.toLowerCase() || "";
  if (props.historic) {
    return listings.value.filter((listing) => listing && listing.name.toLowerCase().includes(searchTerm) && listing.hidden);
  } else {
    if (props.tags && props.tags.length > 0) {
      return listings.value.filter((listing) => listing && listing.name.toLowerCase().includes(searchTerm) && !listing.hidden && itemsWithTag.value?.includes(listing._id.toString()));
    } else {
      return listings.value.filter((listing) => listing && listing.name.toLowerCase().includes(searchTerm) && !listing.hidden);
    }
  }
});

onBeforeMount(async () => {
  await getListings();
  await getTagged();
  loaded.value = true;
});
</script>

<template>
  <section class="thumb-container" v-if="loaded && !tagLoading && filteredListings.length !== 0">
    <article class="thumb" v-for="listing in filteredListings" :key="listing._id">
      <ListingThumbComponent :listingId="listing._id" />
    </article>
  </section>
  <p v-else-if="loaded && !tagLoading">No listings found</p>
  <p v-else>Loading...</p>
</template>

<style scoped>
.thumb-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 280px));
  gap: 2rem;
  padding: 2rem;
  justify-content: start; /* THIS forces left alignment */
}
</style>
