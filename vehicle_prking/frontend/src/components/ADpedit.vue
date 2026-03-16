<template>
  <div class="admin-edit-wrapper">
    <!-- Back button -->
    <div class="top-bar">
      <el-button
        type="primary"
        icon="el-icon-arrow-left"
        class="back-button"
        @click="goBack"
      >
        Back to Home
      </el-button>
    </div>

    <h2>Edit Any User (Admin)</h2>

    <el-select v-model="selectedEmail" placeholder="Select a user" @change="loadUser" class="select-box">
      <el-option
        v-for="user in users"
        :key="user.email"
        :label="user.email"
        :value="user.email"
      />
    </el-select>

    <el-form
      v-if="userLoaded"
      @submit.native.prevent="updateUser"
      label-width="100px"
      class="edit-form"
    >
      <el-form-item label="Full Name">
        <el-input v-model="user.full_name" placeholder="Enter full name" />
      </el-form-item>

      <el-form-item label="Address">
        <el-input v-model="user.address" placeholder="Enter address" />
      </el-form-item>

      <el-form-item label="Pincode">
        <el-input v-model="user.pincode" placeholder="Enter pincode" type="number" />
      </el-form-item>

      <el-form-item>
        <el-button type="success" @click="updateUser">Update</el-button>
      </el-form-item>

      <el-alert
        v-if="message"
        :title="message"
        :type="success ? 'success' : 'error'"
        show-icon
        :closable="false"
        class="message-box"
      />
    </el-form>
  </div>
</template>

<script>
export default {
  name: "AdminEditUser",
  data() {
    return {
      users: [],
      selectedEmail: "",
      user: {
        full_name: "",
        address: "",
        pincode: ""
      },
      message: "",
      success: false,
      userLoaded: false
    };
  },
  mounted() {
    this.fetchAllUsers();
  },
  methods: {
    async fetchAllUsers() {
      const token = localStorage.getItem("access_token");
      try {
        const res = await fetch("http://localhost:5000/api/admin/users", {
          headers: { Authorization: `Bearer ${token}` }
        });
        const data = await res.json();
        if (res.ok) {
          this.users = data;
        } else {
          this.message = data.error || "Failed to fetch users.";
        }
      } catch (err) {
        this.message = "Server error.";
      }
    },
    async loadUser() {
      const token = localStorage.getItem("access_token");
      try {
        const res = await fetch(`http://localhost:5000/api/user/profile/${this.selectedEmail}`, {
          headers: { Authorization: `Bearer ${token}` }
        });
        const data = await res.json();
        if (res.ok) {
          this.user = data;
          this.userLoaded = true;
          this.message = "";
        } else {
          this.message = data.error || "Failed to load user.";
        }
      } catch (err) {
        this.message = "Server error.";
      }
    },
    async updateUser() {
      const token = localStorage.getItem("access_token");
      try {
        const res = await fetch(`http://localhost:5000/api/user/update/${this.selectedEmail}`, {
          method: "PUT",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${token}`
          },
          body: JSON.stringify(this.user)
        });
        const data = await res.json();
        this.message = data.message || data.error;
        this.success = res.ok;
      } catch (err) {
        this.message = "Server error.";
        this.success = false;
      }
    },
    goBack() {
      this.$router.push("/admin");
    }
  }
};
</script>

<style scoped>
.admin-edit-wrapper {
  max-width: 550px;
  margin: 40px auto;
  background: #fff;
  padding: 2rem;
  border-radius: 10px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.top-bar {
  display: flex;
  justify-content: flex-start;
  margin-bottom: 1.5rem;
}

.back-button {
  font-weight: 500;
}

.select-box {
  margin-bottom: 1.5rem;
  width: 100%;
}

.edit-form {
  margin-top: 20px;
}

.message-box {
  margin-top: 1rem;
}
</style>
