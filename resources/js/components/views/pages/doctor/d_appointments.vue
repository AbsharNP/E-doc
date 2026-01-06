<template>
  <div class="doctor-appointments-page">
    <div class="hero glass-panel">
      <div class="hero-text">
        <div class="pill">Schedule</div>
        <h1>Your appointments</h1>
        <p>
          Switch between today’s patients and your full schedule,
          then jump straight into treatment when you’re ready.
        </p>
      </div>
    </div>

    <div class="filters card-elevated">
      <div class="filter-toggle">
        <button
          class="btn pill-btn"
          :class="showToday ? 'btn-primary' : 'btn-outline-primary'"
          @click="fetchTodayAppointments"
        >
          Today
        </button>
        <button
          class="btn pill-btn"
          :class="showAll ? 'btn-primary' : 'btn-outline-primary'"
          @click="fetchAllAppointments"
        >
          All
        </button>
      </div>
    </div>

    <div class="card-elevated table-card">
      <div class="section-heading">Appointments</div>
      <div class="table-shell">
        <div class="table-header-row">
          <span>Patient</span>
          <span>Date</span>
          <span>Status</span>
          <span class="text-end" v-if="showToday">Action</span>
        </div>
        <div v-if="appointments.length === 0" class="empty-row">
          No appointments found.
        </div>
        <div
          v-for="appointment in appointments.slice(0, 5)"
          :key="appointment.id"
          class="table-row"
        >
          <span>{{ appointment.name }}</span>
          <span>{{ appointment.session_date }}</span>
          <span>
            <span
              :class="[
                'status-pill',
                appointment.treatment_status === 1 ? 'done' : 'pending'
              ]"
            >
              {{ appointment.treatment_status === 1 ? 'Completed' : 'Pending' }}
            </span>
          </span>
          <span class="text-end" v-if="showToday">
            <router-link
              :to="`/treatment/${appointment.id}`"
              class="btn btn-primary btn-sm pill-btn"
            >
              Open
            </router-link>
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      appointments: [],
      doctorId: localStorage.getItem('doctorId'),
      showToday: true,
      showAll: false,
    };
  },
  mounted() {
    this.fetchTodayAppointments();  
  },
  methods: {
    fetchTodayAppointments() {
      this.showToday = true;
      this.showAll = false;

      // Fetch only today's appointments
      axios.post(`/appointments/today/${this.doctorId}`)
        .then(response => {
          this.appointments = response.data.map(appointment => ({
            ...appointment,
            session_date: this.formatDate(appointment.session_date)
          }));
        })
        .catch(error => {
          console.error("Error fetching today's appointments:", error);
        });
    },

    fetchAllAppointments() {
      this.showToday = false;
      this.showAll = true;

      // Fetch all appointments
      axios.post(`/appointments/${this.doctorId}`)
        .then(response => {
          this.appointments = response.data
            .map(appointment => ({
              ...appointment,
              session_date: this.formatDate(appointment.session_date)
            }))
            .sort((a, b) => new Date(a.session_date) - new Date(b.session_date));
        })
        .catch(error => {
          console.error("Error fetching all appointments:", error);
        });
    },

    formatDate(dateString) {
      const date = new Date(dateString);
      if (isNaN(date.getTime())) {
        return 'Invalid Date';  // Handle invalid date
      }
      return date.toLocaleDateString('en-GB', { day: '2-digit', month: '2-digit', year: '2-digit' });
    },
  },
};
</script>

<style scoped>
.doctor-appointments-page {
  max-width: 1100px;
  margin: 20px auto 40px;
  padding: 0 18px 32px;
}

.hero {
  padding: 22px 26px;
  border-radius: 24px;
  background: linear-gradient(135deg, #e0f2fe, #f8fbff);
  border: 1px solid rgba(14, 165, 233, 0.2);
  margin-bottom: 18px;
}

.hero-text h1 {
  margin: 8px 0;
  font-size: 24px;
}

.hero-text p {
  color: #475569;
  max-width: 540px;
}

.filters {
  padding: 14px 18px;
  margin-bottom: 18px;
}

.filter-toggle {
  display: inline-flex;
  gap: 8px;
}

.table-card {
  padding: 18px 18px 16px;
}

.table-shell {
  margin-top: 10px;
  border-radius: 18px;
  border: 1px solid #e5e7eb;
  background: #ffffff;
  box-shadow: 0 10px 26px rgba(15, 23, 42, 0.05);
  padding: 10px 16px 12px;
}

.table-header-row,
.table-row {
  display: grid;
  grid-template-columns: 1.6fr 1.2fr 1.1fr 0.9fr;
  gap: 10px;
  align-items: center;
  font-size: 14px;
}

.table-header-row {
  font-weight: 700;
  color: #64748b;
  border-bottom: 1px solid #e5e7eb;
  padding-bottom: 8px;
  margin-bottom: 4px;
}

.table-row {
  padding: 6px 0;
  border-radius: 10px;
}

.table-row:nth-child(odd) {
  background: #f8fafc;
}

.empty-row {
  padding: 12px 0;
  text-align: center;
  color: #94a3b8;
  font-size: 14px;
}

.text-end {
  text-align: right;
}

.status-pill {
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 12px;
  font-weight: 600;
}

.status-pill.done {
  background: #ecfdf3;
  color: #15803d;
}

.status-pill.pending {
  background: #fef9c3;
  color: #92400e;
}

.pill-btn {
  border-radius: 999px;
  font-weight: 600;
}

@media (max-width: 768px) {
  .table-header-row,
  .table-row {
    grid-template-columns: 1.4fr 1.3fr 1.1fr 0.9fr;
    font-size: 13px;
  }
}
</style>
