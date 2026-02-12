<script setup lang="ts">
import { fetchy } from "@/utils/fetchy";
import { ref } from "vue";
import noImage from "@/assets/images/no-image.jpg";


import { useRouter } from "vue-router";


const props = defineProps(["requestId"]);

const imageUrl = ref("");
const meetupLocation = ref("");
const message = ref("");

const imageError = ref("");

const isValidUrl = (url: string) => {
  try {
    const u = new URL(url);
    return u.protocol === "http:" || u.protocol === "https:";
  } catch {
    return false;
  }
};

const router = useRouter();

const makeOffer = async () => {
  imageError.value = "";
  if (!meetupLocation.value) {
    alert("Meetup location is required.");
    return;
  }
  if (imageUrl.value && !isValidUrl(imageUrl.value)) {
    imageError.value = "Please enter a valid image URL.";
    return;
  }
  try {
    const requestId = props.requestId;
    await fetchy("/api/offers", "POST", {
      body: {
        requestId,
        location: meetupLocation.value,
        image: imageUrl.value,
        message: message.value,
      },
    });
    // Reset form fields
    meetupLocation.value = "";
    imageUrl.value = "";
    message.value = "";
    await router.push(`/requests/${props.requestId}`);
  } catch {
    alert("There was an error making the offer.");
  }
};

function goBack() {
  void router.push(`/requests/${props.requestId}`);
}
</script>

<template>
  <button type="button" @click="goBack" class="cancel-top">Cancel</button>
  <div class="form-container">
    <div class="image-container">
        <img :src="imageUrl && isValidUrl(imageUrl) ? imageUrl : noImage" />
      </div>
    <form @submit.prevent="makeOffer" class="pure-form pure-form-stacked create-offer-form">
      
      <h1>Offer</h1>
      <p>Once you make the offer and it is accepted by the requester, they will be able to contact you by your contact info.</p>
      <label for="imageUrl">Image URL</label>
      <input id="imageUrl" type="url" v-model="imageUrl" placeholder="https://example.com/image.jpg" />
      <small v-if="imageError" style="color: #c72d12">{{ imageError }}</small>

      <label for="meetupLocation"> <span style="font-size: 25px">&#128205;</span>Meet Up Location </label>
      <input id="meetupLocation" type="text" v-model="meetupLocation" placeholder="Meetup Location" required />

      <label for="message">Message</label>
      <input id="message" type="text" v-model="message" placeholder="Message (optional)" />
      
      <button type="submit" class="pure-button pure-button-primary">Send Offer</button>
    </form>
  </div>
</template>

<style scoped>
.form-container {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 2em;
  padding: 1em;
}

.create-offer-form {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.5em;
}

button {
  background-color: #69825a;
  border-radius: 10px;
}

.image-container {
  width: 400px;
  height: 400px;
  flex-shrink: 0;
  border-radius: 10px;
  overflow: hidden;
  background-color: #f3f3f3;
  display: flex;
  align-items: center;
  justify-content: center;
}

.image-container img {
  width: 100%;
  height: 100%;
  border-radius: 10px;
  object-fit: cover;
  border: 1px solid #ccc;
}
/* Style for the cancel button at the top */
.cancel-top {
  align-self: flex-end;
  margin-bottom: 1em;
  background: #c72d12;
  color: #fff;
  border: none;
  padding: 0.5em 1.2em;
  font-size: 1em;
  border-radius: 8px;
  cursor: pointer;
}
</style>