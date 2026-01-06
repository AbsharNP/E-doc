<template>
  <div class="nav-wrapper">
    <nav class="navbar navbar-expand-lg glass-nav fixed-top">
      <div class="container align-items-center">
        <a class="brand d-flex align-items-center" href="#">
          <span class="logo-pill">E</span>
          <span class="ms-2 fw-bold">Edoc</span>
        </a>
        <button
          class="navbar-toggler"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#navbarResponsive"
          aria-controls="navbarResponsive"
          aria-expanded="false"
          aria-label="Toggle navigation"
        >
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarResponsive">
          <ul class="navbar-nav me-auto">
            <li class="nav-item" v-if="userType === 'admin'">
              <a class="nav-link" href="/admin">Home</a>
            </li>
            <li class="nav-item" v-if="userType === 'patient'">
              <a class="nav-link" href="/patient-dash">Home</a>
            </li>
            <li class="nav-item" v-if="userType === 'doctor'">
              <a class="nav-link" href="/doctor-dash">Home</a>
            </li>
          </ul>
          <ul class="navbar-nav ms-auto align-items-center gap-2">
            <li class="nav-item" v-if="userType === 'doctor'">
              <a class="nav-link pill-link" href="/appointment">Appointments</a>
            </li>
            <li class="nav-item" v-if="userType === 'patient'">
              <a class="nav-link pill-link" href="/view-prescriptions">History</a>
            </li>
            <li class="nav-item" v-if="userType === 'patient'">
              <a class="nav-link profile-link d-flex align-items-center" href="/profile">
                <img
                  alt="avatar"
                  :src="profilePic"
                  class="small-avatar me-2"
                />
                <span class="d-none d-lg-inline">Profile</span>
              </a>
            </li>
            <li class="nav-item">
              <a class="btn btn-sm btn-primary" href="#" @click="logout">
                Logout
              </a>
            </li>
          </ul>
        </div>
      </div>
    </nav>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'NavBar',
  data() {
    return {
      userType: null,
      profilePic: null, // Initialize profilePic to null
      userId: localStorage.getItem('userId'), // Get user ID from localStorage
    };
  },
  mounted() {
    // Get userType from localStorage when the component is mounted
    this.userType = localStorage.getItem('userType');
    this.getProfilePic(); // Call function to get profile picture
  },
  methods: {
    getProfilePic() {
      axios
        .post(`/getProfilePic/${this.userId}`)
        .then((response) => {
          this.profilePic = response.data.profile_pic;
        })
        .catch((error) => {
          console.error('Error fetching profile picture:', error);
          this.profilePic = '/images/default-avatar.jpg'; // Fallback to default avatar
        });
    },
    // Function to handle user logout
    async logout() {
      try {
        const response = await axios.post('/logout');

        if (response.status === 200) {
          // Clear all local storage items
          localStorage.removeItem('authToken');
          localStorage.removeItem('userName');
          localStorage.removeItem('userEmail');
          localStorage.removeItem('isAuthenticated');
          localStorage.removeItem('userId');
          localStorage.removeItem('userType');
          localStorage.removeItem('profile_pic');
          localStorage.removeItem('doctorId');

          this.$router.push({ name: 'login' });
          this.$toast.success('Logged out successfully');
        } else {
          this.$toast.error('Error logging out. Please try again.');
        }
      } catch (error) {
        console.error('Error logging out:', error);
        this.$toast.error('Error logging out');
      }
    },
  },
};
</script>

<style scoped>
.nav-wrapper {
  backdrop-filter: blur(6px);
}

.glass-nav {
  background: rgba(255, 255, 255, 0.92);
  border-bottom: 1px solid rgba(229, 231, 235, 0.7);
  box-shadow: 0 10px 40px rgba(14, 165, 233, 0.08);
  border-radius: 16px;
  margin: 10px 16px;
  padding: 8px 18px;
}

.brand {
  color: #0f172a;
  font-size: 18px;
}

.logo-pill {
  width: 34px;
  height: 34px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #0ea5e9, #0284c7);
  color: #fff;
  border-radius: 12px;
  font-weight: 800;
  box-shadow: 0 10px 30px rgba(14, 165, 233, 0.35);
}

.nav-link {
  font-weight: 600;
  color: #0f172a;
  padding: 8px 12px;
  border-radius: 12px;
}

.nav-link:hover {
  background: #e0f2fe;
  color: #0284c7;
}

.pill-link {
  background: #e0f2fe;
  color: #0284c7;
  border-radius: 999px;
  padding: 8px 14px;
  font-weight: 700;
}

.profile-link {
  background: rgba(14, 165, 233, 0.08);
  border-radius: 999px;
  padding: 6px 10px;
  color: #0f172a;
}

.small-avatar {
  width: 34px;
  height: 34px;
  object-fit: cover;
  border-radius: 50%;
  border: 2px solid #e0f2fe;
}

.btn-primary {
  padding: 8px 14px;
}

@media (max-width: 992px) {
  .glass-nav {
    margin: 0;
    border-radius: 0;
  }
}
</style>
