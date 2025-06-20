<template>
  <div>
    <h1>Welcome to Vue Practice</h1>
    <button @click="openAddDrawer">Add User</button>

    <input placeholder="Search" v-model="searchInput" @input="onSearch" />

    <table border="1" cellpadding="8" cellspacing="0">
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Username</th>
          <th>Email</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in filteredData" :key="index">
          <td>{{ item.id }}</td>
          <td>{{ item.name }}</td>
          <td>{{ item.username }}</td>
          <td>{{ item.email }}</td>
          <td>
            <button @click="deleteUser(item.id)">Delete</button>
            <button @click="openEditDrawer(item)">Update</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Drawer (Add/Edit User) -->
    <div v-if="isDrawerOpen" class="drawer">
      <h3>{{ isEditMode ? "Update User" : "Add User" }}</h3>
      <form @submit.prevent="isEditMode ? updateUser() : submitUser()">
        <input v-model="newUser.name" placeholder="Name" required /><br />
        <input
          v-model="newUser.username"
          placeholder="Username"
          required
        /><br />
        <input v-model="newUser.email" placeholder="Email" required /><br />
        <button type="submit">{{ isEditMode ? "Update" : "Submit" }}</button>
        <button type="button" @click="closeDrawer">Cancel</button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import debounce from "lodash/debounce";

const searchInput = ref("");
const users = ref([]);
const filteredData = ref([]);

const isDrawerOpen = ref(false);
const isEditMode = ref(false);
const editUserId = ref(null);

const newUser = ref({
  name: "",
  username: "",
  email: "",
});

// Fetch Users
const usersInfo = async () => {
  const response = await axios.get(
    "https://jsonplaceholder.typicode.com/users"
  );
  users.value = response.data;
  filteredData.value = users.value;
};
usersInfo();

// Search Filter
const refilter = () => {
  const query = searchInput.value.toLowerCase();
  if (!query) {
    filteredData.value = users.value;
  } else {
    filteredData.value = users.value.filter((item) =>
      item.name.toLowerCase().includes(query)
    );
  }
};
const onSearch = debounce(refilter, 500);

// Open drawer for Add
const openAddDrawer = () => {
  isEditMode.value = false;
  newUser.value = { name: "", username: "", email: "" };
  isDrawerOpen.value = true;
};

// Open drawer for Edit
const openEditDrawer = (user) => {
  isEditMode.value = true;
  editUserId.value = user.id;
  newUser.value = { ...user }; // Prefill
  isDrawerOpen.value = true;
};

// Close drawer
const closeDrawer = () => {
  isDrawerOpen.value = false;
  newUser.value = { name: "", username: "", email: "" };
  editUserId.value = null;
};

// Delete user
const deleteUser = async (id) => {
  try {
    const res = await axios.delete(
      `https://jsonplaceholder.typicode.com/users/${id}`
    );
    if (res.status === 200) {
      users.value = users.value.filter((user) => user.id !== id);
      filteredData.value = filteredData.value.filter((user) => user.id !== id);
      console.log(`User ${id} deleted (fake)`);
    }
  } catch (error) {
    console.error("Failed to delete user:", error);
  }
};

// Add user
const submitUser = async () => {
  try {
    const res = await axios.post(
      "https://jsonplaceholder.typicode.com/users",
      newUser.value
    );
    if (res.status === 201) {
      const addedUser = { ...res.data, id: Date.now() };
      users.value.push(addedUser);
      filteredData.value.push(addedUser);
      closeDrawer();
      console.log("User added:", addedUser);
    }
  } catch (error) {
    console.error("Failed to add user:", error);
  }
};

// Update user
const updateUser = async () => {
  try {
    const res = await axios.put(
      `https://jsonplaceholder.typicode.com/users/${editUserId.value}`,
      newUser.value
    );
    if (res.status === 200) {
      const updatedUser = { ...res.data, id: editUserId.value };

      // Update in local users and filteredData
      const updateInArray = (arr) => {
        const index = arr.findIndex((user) => user.id === editUserId.value);
        if (index !== -1) arr[index] = updatedUser;
      };
      updateInArray(users.value);
      updateInArray(filteredData.value);

      closeDrawer();
      console.log("User updated:", updatedUser);
    }
  } catch (error) {
    console.error("Failed to update user:", error);
  }
};
</script>

<style scoped>
.drawer {
  position: fixed;
  top: 0;
  right: 0;
  width: 300px;
  height: 100%;
  background: #f4f4f4;
  padding: 20px;
  box-shadow: -2px 0 5px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}
</style>
