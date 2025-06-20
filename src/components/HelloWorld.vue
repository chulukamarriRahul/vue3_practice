<template>
  <div>
    <h1>Welcome to Vue Practice</h1>

    <div class="button-container">
      <button class="add-user-btn" @click="openAddDrawer">Add User</button>
    </div>

    <div class="search-bar">
      <input
        class="search-input"
        placeholder="Search By Name"
        v-model="searchInput"
        @input="onSearch"
      />
    </div>
    <div class="table-container">
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
              <button class="btn btn-danger" @click="deleteUser(item.id)">
                Delete
              </button>
              <button class="btn btn-primary" @click="openEditDrawer(item)">
                Update
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

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
        <button type="submit" class="btn btn-primary">
          {{ isEditMode ? "Update" : "Submit" }}
        </button>
        <button type="button" class="btn btn-danger" @click="closeDrawer">
          Cancel
        </button>
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
      const addedUser = { ...res.data };
      filteredData.value.push(addedUser);
      closeDrawer();
      console.log("User added:", addedUser);
    }
  } catch (error) {
    console.error("Failed to add user:", error);
  }
};

// Update user
const updateUser = () => {
  alert("hi");
  const updatedUser = { ...newUser.value, id: editUserId.value };

  // Update users
  users.value = users.value.map((user) =>
    user.id === editUserId.value ? updatedUser : user
  );

  // Update filteredData
  filteredData.value = filteredData.value.map((user) =>
    user.id === editUserId.value ? updatedUser : user
  );

  closeDrawer();
  console.log("User updated:", updatedUser);
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

.table-container {
  height: 100vh;
  overflow: auto;
  width: 100%;
}

table {
  width: 100%;
  border-collapse: collapse;
}
th,
td {
  text-align: left;
  padding: 8px;
}

.button-container {
  text-align: right;
  margin-bottom: 10px;
}

.add-user-btn {
  background-color: #007bff; /* Bootstrap blue */
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}

.add-user-btn:hover {
  background-color: #0056b3;
}
.btn {
  padding: 8px 14px;
  margin-right: 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* Blue Buttons */
.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-primary:hover {
  background-color: #0056b3;
}

/* Red Buttons */
.btn-danger {
  background-color: #dc3545;
  color: white;
}

.btn-danger:hover {
  background-color: #b52a37;
}

.search-bar {
  margin-bottom: 20px;
}

.search-input {
  width: 300px;
  padding: 8px 12px;
  border-radius: 4px;
  border: 1px solid #ccc;
  font-size: 14px;
}
</style>
