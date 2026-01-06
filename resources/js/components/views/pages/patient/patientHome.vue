<template>
  <div class="page">
    <div class="hero glass-panel">
      <div class="hero-text">
        <div class="pill">Hi {{ userName || 'there' }}</div>
        <h1>Book, track, and manage your care effortlessly</h1>
        <p>
          Jump into the booking wizard to find the right doctor, secure a slot,
          and keep upcoming visits organized.
        </p>
        <div class="hero-actions">
          <router-link to="/doc-list" class="btn btn-primary">Start booking</router-link>
          <router-link to="/view-prescriptions" class="btn btn-outline-primary ms-2">View history</router-link>
        </div>
      </div>
      <div class="hero-meta">
        <div class="date-chip">Today · {{ todayDate }}</div>
        <img v-if="profilePic" :src="profilePic" alt="profile" class="hero-avatar" />
      </div>
    </div>

    <div class="layout-grid">
      <div class="card-elevated wizard-card">
        <div class="section-heading">Book an appointment</div>
        <div class="wizard-steps">
          <div class="step">
            <div class="step-badge">1</div>
            <div>
              <h4>Choose a doctor</h4>
              <p>Browse specialties and availability.</p>
              <router-link to="/doc-list" class="link">Find doctor</router-link>
            </div>
          </div>
          <div class="step">
            <div class="step-badge">2</div>
            <div>
              <h4>Select a session</h4>
              <p>Pick a slot that fits your schedule.</p>
              <router-link to="/available-sessions" class="link">Available sessions</router-link>
            </div>
          </div>
          <div class="step">
            <div class="step-badge">3</div>
            <div>
              <h4>Confirm & track</h4>
              <p>Get reminders and access visit details.</p>
              <router-link to="/patient-dash" class="link">My dashboard</router-link>
            </div>
          </div>
        </div>
      </div>

      <div class="card-elevated appointments-card">
        <div class="section-heading">Upcoming appointments</div>
        <div v-if="error" class="alert alert-danger">{{ error }}</div>
        <div v-if="appointments.length === 0" class="empty-state">
          <p>No appointments yet. Book your first visit.</p>
          <router-link to="/doc-list" class="btn btn-primary btn-sm">Book now</router-link>
        </div>
        <div class="appointments-grid">
          <div
            class="appointment-tile"
            v-for="appointment in appointments"
            :key="appointment.id"
          >
            <div class="tile-header">
              <div>
                <p class="label">Doctor</p>
                <h4>{{ capitalizeWords(appointment.dr_name) }}</h4>
              </div>
              <span :class="['status', appointment.treatment_status ? 'success' : 'pending']">
                {{ appointment.treatment_status ? 'Completed' : 'Scheduled' }}
              </span>
            </div>
            <div class="tile-body">
              <div>
                <p class="label">Date</p>
                <p class="value">{{ formatDate(appointment.appsession.session_date) }}</p>
              </div>
              <div>
                <p class="label">Patient</p>
                <p class="value">{{ capitalizeWords(appointment.name) }}</p>
              </div>
            </div>
            <div class="tile-actions">
              <router-link
                v-if="appointment.treatment_status === 1"
                :to="`/view-prescriptions`"
                class="btn btn-outline-primary btn-sm"
              >
                View prescription
              </router-link>
              <button
                v-else
                class="btn btn-outline-danger btn-sm"
                @click="deleteAppointment(appointment.id)"
              >
                Cancel
              </button>
            </div>
          </div>
        </div>
      </div>

      <div class="card-elevated quick-links">
        <div class="section-heading">Quick actions</div>
        <div class="quick-grid">
          <router-link
            v-for="card in cards"
            :key="card.title"
            :to="card.link"
            class="quick-card"
          >
            <h4>{{ card.title }}</h4>
            <p>{{ card.description }}</p>
          </router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Swal from 'sweetalert2';

export default {
  data() {
    return {
      todayDate: new Date().toISOString().split('T')[0],
      userName: localStorage.getItem('userName'),
      profilePic: '',
      cards: [
        { title: 'Get Appointment', description: 'Book your appointment now', link: '/doc-list' },
        // { title: 'My Appointments', description: 'Track all your current appointments', link: '/my-appointment' },
        // { title: 'Prescriptions', description: 'Check your prescribed medications', link: '/view-prescriptions' },
      ],
      appointments: [],
      error: null,
    };
  },
  mounted() {
    this.fetchAppointments();
    this.getProfilePic(); // Call the method to fetch the profile picture
  },
  methods: {
    async fetchAppointments() {
      try {
        const userId = localStorage.getItem('userId'); // Get user ID from local storage
        const response = await axios.post('/my-appointments', { user_id: userId }); // Send user ID in request body
        this.appointments = response.data.sort((a, b) => {
          const dateA = new Date(a.appsession.session_date);
          const dateB = new Date(b.appsession.session_date);
          return dateA - dateB; // Sort in ascending order (oldest first) 
        });
      } catch (error) {
        console.error('Error fetching appointments:', error.response?.data || error);
        this.error = 'Failed to load appointments.';
      }
    },

    async getProfilePic() {
      const userId = localStorage.getItem('userId'); // Get user ID from local storage
      try {
        const response = await axios.post(`/getProfilePic/${userId}`);
        this.profilePic = response.data.profile_pic || '/images/default-avatar.jpg'; // Fallback to default avatar if no picture
      } catch (error) {
        console.error('Error fetching profile picture:', error);
        this.profilePic = '/images/default-avatar.jpg'; // Fallback to default avatar on error
      }
    },

    async deleteAppointment(id) {
      const result = await Swal.fire({
        title: 'Are you sure?',
        text: "This action cannot be undone.",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#d33',
        cancelButtonColor: '#3085d6',
        confirmButtonText: 'Yes, delete it!',
      });

      if (result.isConfirmed) {
        try {
          await axios.delete(`/appointments/${id}`);
          this.appointments = this.appointments.filter(appointment => appointment.id !== id);
          Swal.fire('Deleted!', 'Your appointment has been deleted.', 'success');
        } catch (error) {
          console.error('Error deleting appointment:', error.response?.data || error);
          this.error = 'Failed to delete appointment.';
          Swal.fire('Error!', 'There was a problem deleting the appointment.', 'error');
        }
      }
    },
    formatDate(dateString) {
      const options = { year: 'numeric', month: 'long', day: 'numeric' };
      return new Date(dateString).toLocaleDateString(undefined, options);
    },
    capitalizeWords(str) {
      return str.split(' ').map(word =>
        word.charAt(0).toUpperCase() + word.slice(1).toLowerCase()
      ).join(' ');
    },
  },
};
</script>

<style scoped>
.page {
  max-width: 1200px;
  margin: 20px auto 40px;
  padding: 0 18px 32px;
}

.hero {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 28px;
  border-radius: 24px;
  background: linear-gradient(135deg, #e0f2fe, #f8fbff);
  border: 1px solid rgba(14, 165, 233, 0.2);
}

.hero-text h1 {
  margin: 10px 0;
  font-size: 28px;
}

.hero-text p {
  color: #475569;
  max-width: 520px;
}

.hero-meta {
  display: flex;
  align-items: center;
  gap: 12px;
}

.date-chip {
  padding: 10px 14px;
  border-radius: 14px;
  background: #0ea5e9;
  color: #fff;
  font-weight: 700;
}

.hero-avatar {
  width: 72px;
  height: 72px;
  border-radius: 50%;
  object-fit: cover;
  border: 3px solid #e0f2fe;
  box-shadow: 0 12px 30px rgba(14, 165, 233, 0.25);
}

.hero-actions {
  margin-top: 12px;
}

.layout-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 20px;
  margin-top: 20px;
}

.wizard-card {
  padding: 22px;
}

.wizard-steps {
  display: grid;
  gap: 14px;
  margin-top: 16px;
}

.step {
  display: grid;
  grid-template-columns: 56px 1fr;
  gap: 12px;
  align-items: center;
  padding: 14px;
  border: 1px dashed rgba(14, 165, 233, 0.2);
  border-radius: 16px;
  background: #f8fbff;
}

.step-badge {
  width: 50px;
  height: 50px;
  border-radius: 18px;
  display: grid;
  place-items: center;
  background: #0ea5e9;
  color: #fff;
  font-weight: 800;
  box-shadow: 0 10px 30px rgba(14, 165, 233, 0.2);
}

.step h4 {
  margin: 0;
}

.link {
  font-weight: 700;
  color: #0ea5e9;
}

.appointments-card {
  padding: 22px;
}

.appointments-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 14px;
  margin-top: 16px;
}

.appointment-tile {
  padding: 16px;
  border-radius: 18px;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  box-shadow: 0 12px 32px rgba(15, 23, 42, 0.05);
}

.tile-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.status {
  padding: 6px 12px;
  border-radius: 999px;
  font-weight: 700;
  font-size: 12px;
}

.status.success {
  background: #ecfdf3;
  color: #15803d;
}

.status.pending {
  background: #e0f2fe;
  color: #075985;
}

.tile-body {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
  margin: 12px 0;
}

.label {
  color: #94a3b8;
  margin: 0;
}

.value {
  margin: 2px 0 0;
  font-weight: 700;
}

.tile-actions {
  display: flex;
  justify-content: space-between;
  gap: 8px;
}

.quick-links {
  padding: 22px;
}

.quick-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 12px;
  margin-top: 14px;
}

.quick-card {
  padding: 16px;
  border-radius: 16px;
  border: 1px solid #e5e7eb;
  background: linear-gradient(135deg, #ffffff, #f8fbff);
  box-shadow: 0 10px 24px rgba(14, 165, 233, 0.08);
  color: #0f172a;
}

.empty-state {
  text-align: center;
  color: #94a3b8;
  padding: 20px 0;
}

@media (max-width: 768px) {
  .hero {
    flex-direction: column;
    align-items: flex-start;
    gap: 16px;
  }

  .tile-body {
    grid-template-columns: 1fr;
  }
}
</style>
