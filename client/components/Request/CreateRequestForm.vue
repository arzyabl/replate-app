<script setup lang="ts">
import { fetchy } from "@/utils/fetchy"; // Ensure this is set up to handle your API requests
import { ref } from "vue";
import { useRouter } from "vue-router";
import noImage from "@/assets/images/no-image.jpg";

// Form fields
const name = ref("");
const imageUrl = ref("");
const quantity = ref(0);
const needBy = ref("");
const description = ref("");

// Validation helpers
const imageError = ref("");
const dateError = ref("");
const quantityError = ref("");

const isValidUrl = (url: string) => {
  try {
    const u = new URL(url);
    return u.protocol === "http:" || u.protocol === "https:";
  } catch {
    return false;
  }
};
const isValidDate = (value: string) => {
  if (!value) return false;
  const d = new Date(value);
  return !isNaN(d.getTime());
};

// Use Vue Router
const router = useRouter();

// Create a new Request
const createRequest = async () => {
  imageError.value = "";
  dateError.value = "";
  quantityError.value = "";

  if (!name.value || !quantity.value || !needBy.value || !description.value) {
    alert("All fields are required.");
    return;
  }

  if (imageUrl.value && !isValidUrl(imageUrl.value)) {
    imageError.value = "Please enter a valid image URL.";
    return;
  }
  if (!isValidDate(needBy.value)) {
    dateError.value = "Please select a valid date.";
    return;
  }
  if (quantity.value < 1 || quantity.value > 100) {
    quantityError.value = "Quantity must be between 1 and 100.";
    return;
  }

  try {
    // Make API call to create a Request
    const response = await fetchy("/api/requests", "POST", {
      body: {
        name: name.value,
        quantity: quantity.value,
        needBy: needBy.value,
        ...(imageUrl.value ? { image: imageUrl.value } : {}),
        description: description.value,
      },
    });
    if (response.msg) {
      alert("Request created successfully!");
      // Reset form fields
      name.value = "";
      quantity.value = 0;
      needBy.value = "";
      description.value = "";
      // Navigate back to the home view
      await router.push({ name: "Home" });
    } else {
      throw new Error("Failed to create the Request.");
    }
  } catch (error) {
    console.error("Error creating Request:", error);
    alert("There was an error creating the Request.");
  }
};
</script>

<template>
  <div class="form-container">
    <div class="image-container">
      <img :src="imageUrl && isValidUrl(imageUrl) ? imageUrl : noImage" />
    </div>
    <form @submit.prevent="createRequest" class="pure-form pure-form-stacked create-request-form">
      <label for="name">Item Name</label>
      <input id="name" type="text" v-model="name" placeholder="Name" required />

      <label for="imageUrl">Image URL (optional)</label>
      <input id="imageUrl" type="url" v-model="imageUrl" placeholder="https://example.com/image.jpg" />
      <small v-if="imageError" style="color: #c72d12">{{ imageError }}</small>

      <label for="quantity">Quantity</label>
      <input id="quantity" type="number" v-model.number="quantity" placeholder="1" required min="1" max="100" />
      <small v-if="quantityError" style="color: #c72d12">{{ quantityError }}</small>

      <label for="needBy">Need By</label>
      <input id="needBy" type="date" v-model="needBy" required />
      <small v-if="dateError" style="color: #c72d12">{{ dateError }}</small>

      <label for="description">Description</label>
      <input id="description" v-model="description" placeholder="Description" required />

      <div class="faq-link">
        <p>
          Have questions?
          <RouterLink to="/faq" class="faq-anchor">Visit our FAQ</RouterLink>
        </p>
      </div>
      <button type="submit" class="pure-button pure-button-primary">Create Request</button>
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

.create-request-form {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.5em;
}

input {
  padding: 0.5em 0.6em;
  display: inline-block;
  border: 1px solid #ccc;
  box-shadow: inset 0 1px 3px #ddd;
  border-radius: 4px;
  vertical-align: middle;
  box-sizing: border-box;
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
</style>
