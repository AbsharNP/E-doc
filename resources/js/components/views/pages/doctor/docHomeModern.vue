<template>
  <div class="doctor-page">
    <!-- Top hero / header -->
    <div class="hero glass-panel">
      <div class="hero-text">
        <div class="pill">Doctor workspace</div>
        <h1>Good day, Dr. {{ capitalizedUserName }}</h1>
        <p>
          Review today’s schedule at a glance, manage your time blocks,
          and jump into patient appointments in one place.
        </p>
        <router-link to="/appointment" class="btn btn-primary">
          View today’s appointments
        </router-link>
      </div>
      <div class="hero-meta">
        <div class="date-chip">Today · {{ todayDate }}</div>
      </div>
    </div>

    <!-- Main grid -->
    <div class="layout-grid">
      <!-- Daily schedule card -->
      <div class="card-elevated schedule-card">
        <div class="section-heading">Upcoming sessions</div>
        <div class="schedule-meta">
          <div>
            <p class="label">Total sessions</p>
            <p class="value">{{ sessions.length }}</p>
          </div>
        </div>
        <div class="table-shell" v-if="sessions.length === 0">
          <p class="empty-copy">
            You don’t have any sessions scheduled yet. Create your first time block to start accepting bookings.
          </p>
          <button
            class="btn btn-outline-primary btn-sm"
            data-bs-toggle="modal"
            data-bs-target="#createSessionModal"
          >
            Add new session
          </button>
        </div>
        <div v-else class="table-shell">
          <div class="table-header-row">
            <span>#</span>
            <span>Scheduled date</span>
            <span>Start</span>
            <span>End</span>
          </div>
          <div
            class="table-row"
            v-for="(session, index) in sessions"
            :key="index"
          >
            <span>{{ index + 1 }}</span>
            <span>{{ formatDate(session.session_date) }}</span>
            <span>{{ session.start_time }}</span>
            <span>{{ session.end_time }}</span>
          </div>
        </div>
      </div>

      <!-- Session management / quick actions -->
      <div class="card-elevated manage-card">
        <div class="section-heading">Manage sessions</div>
        <p class="muted">
          Define when you are available, and we’ll keep your patients in sync automatically.
        </p>

        <div class="today-session">
          <div v-if="todaySession">
            <p class="label">Today’s session</p>
            <p class="value">
              {{ formatDate(todaySession.session_date) }} ·
              {{ todaySession.start_time }} – {{ todaySession.end_time }}
            </p>
          </div>
          <div v-else>
            <p class="label">Today’s session</p>
            <p class="muted small">
              No session created for today yet. Create one to accept bookings.
            </p>
          </div>
        </div>

        <button
          class="btn btn-primary w-100 mb-2 pill-btn"
          data-bs-toggle="modal"
          data-bs-target="#createSessionModal"
        >
          Add today’s session
        </button>
        <button class="btn btn-outline-primary w-100 pill-btn" @click="$router.push('/appointment')">
          Open appointments table
        </button>
      </div>
    </div>

    <!-- Modal for creating a session -->
    <div
      class="modal fade"
      id="createSessionModal"
      tabindex="-1"
      aria-labelledby="createSessionLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="createSessionLabel">Add New Session</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="createSession">
              <div class="mb-3">
                <label for="sessionDate" class="form-label">Session Date</label>
                <input
                  ref="sessionDateInput"
                  type="text"
                  class="form-control"
                  id="sessionDate"
                  :value="form.session_date"
                  placeholder="Select a date"
                  readonly
                  required
                />
              </div>
              <div class="mb-3">
                <label for="startTime" class="form-label">Start Time</label>
                <input
                  ref="startTimeInput"
                  type="text"
                  class="form-control"
                  id="startTime"
                  :value="form.start_time"
                  placeholder="Select start time"
                  readonly
                  required
                />
              </div>
              <div class="mb-3">
                <label for="endTime" class="form-label">End Time</label>
                <input
                  ref="endTimeInput"
                  type="text"
                  class="form-control"
                  id="endTime"
                  :value="form.end_time"
                  placeholder="Select end time"
                  readonly
                  required
                />
              </div>
              <div class="mb-3">
                <label for="numberOfSessions" class="form-label"
                  >Number of Sessions</label
                >
                <input
                  type="number"
                  class="form-control"
                  id="numberOfSessions"
                  v-model="form.no_of_sessions"
                  readonly
                />
              </div>
              <button type="submit" class="btn btn-primary">Create Session</button>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import moment from 'moment';
import Swal from 'sweetalert2';
import flatpickr from 'flatpickr';
import 'flatpickr/dist/flatpickr.css';

export default {
  data() {
    return {
      todayDate: new Date().toISOString().split('T')[0],
      sessions: [],
      userName: localStorage.getItem('userName'),
      form: {
        session_date: '',
        start_time: '',
        end_time: '',
        doctor_id: '',
        no_of_sessions: 0,
      },
      duration: 0,
      datePicker: null,
      startTimePicker: null,
      endTimePicker: null,
    };
  },
  computed: {
    capitalizedUserName() {
      return this.userName
        ? this.userName.charAt(0).toUpperCase() + this.userName.slice(1)
        : 'User';
    },
    todaySession() {
      if (!this.sessions.length) return null;
      const today = this.todayDate;

      return (
        this.sessions.find((s) => {
          if (!s.session_date) return false;
          // Normalize to YYYY-MM-DD string if needed
          const dateStr =
            typeof s.session_date === 'string'
              ? s.session_date.substring(0, 10)
              : new Date(s.session_date).toISOString().substring(0, 10);
          return dateStr === today;
        }) || null
      );
    },
  },
  mounted() {
    const doctorId = localStorage.getItem('doctorId');
    const dr_name = localStorage.getItem('userName');

    if (doctorId && dr_name) {
      this.form.doctor_id = doctorId;
      this.form.dr_name = dr_name;
      this.fetchUpcomingSessions();
    } else {
      Swal.fire({
        icon: 'error',
        title: 'Error',
        text: 'Doctor ID not found. Please log in again.',
      });
      this.$router.push('/login');
    }

    this.initPickers();
  },
  methods: {
    fetchUpcomingSessions() {
      axios
        .post(`/upcsessions/${this.form.doctor_id}`)
        .then((response) => {
          this.sessions = response.data;
        })
        .catch((error) => {
          console.error('Error fetching upcoming sessions:', error);
        });
    },
    initPickers() {
      // Date picker
      if (this.$refs.sessionDateInput) {
        this.datePicker = flatpickr(this.$refs.sessionDateInput, {
          dateFormat: 'Y-m-d',
          defaultDate: this.form.session_date || null,
          onChange: (selectedDates, dateStr) => {
            this.form.session_date = dateStr;
          },
        });
      }

      // Start time picker
      if (this.$refs.startTimeInput) {
        this.startTimePicker = flatpickr(this.$refs.startTimeInput, {
          enableTime: true,
          noCalendar: true,
          dateFormat: 'H:i',
          time_24hr: false,
          onChange: (selectedDates, timeStr) => {
            this.form.start_time = timeStr;
            this.calculateDuration();
          },
        });
      }

      // End time picker
      if (this.$refs.endTimeInput) {
        this.endTimePicker = flatpickr(this.$refs.endTimeInput, {
          enableTime: true,
          noCalendar: true,
          dateFormat: 'H:i',
          time_24hr: false,
          onChange: (selectedDates, timeStr) => {
            this.form.end_time = timeStr;
            this.calculateDuration();
          },
        });
      }
    },
    calculateDuration() {
      const start = moment(this.form.start_time, 'HH:mm');
      const end = moment(this.form.end_time, 'HH:mm');

      if (!start.isValid() || !end.isValid()) {
        this.duration = 0;
        this.form.no_of_sessions = 0;
        return;
      }

      let diffInHours = end.diff(start, 'hours');
      if (diffInHours < 0) {
        end.add(1, 'day');
        diffInHours = end.diff(start, 'hours');
      }

      this.duration = diffInHours;
      this.form.no_of_sessions = this.duration * 3; // Assuming each session is 20 minutes
    },
    createSession() {
      axios
        .post('/create-sessions', this.form)
        .then(() => {
          Swal.fire({
            icon: 'success',
            title: 'Session created successfully',
            toast: true,
            position: 'top-end',
            showConfirmButton: false,
            timer: 2000,
          });
          this.resetForm();
          const modal = new bootstrap.Modal(
            document.getElementById('createSessionModal')
          );
          modal.hide();
          window.location.reload();
        })
        .catch((error) => {
          Swal.fire({
            icon: 'error',
            title: 'Creation failed',
            text:
              error.response?.data?.message ||
              'An error occurred. Please try again.',
          });
        });
    },
    resetForm() {
      this.form.session_date = '';
      this.form.start_time = '';
      this.form.end_time = '';
      this.form.no_of_sessions = 0;
      this.duration = 0;

      if (this.datePicker) this.datePicker.clear();
      if (this.startTimePicker) this.startTimePicker.clear();
      if (this.endTimePicker) this.endTimePicker.clear();
    },
    formatDate(date) {
      return moment(date).format('MMMM Do YYYY');
    },
  },
};
</script>

<style scoped>
.doctor-page {
  max-width: 1200px;
  margin: 20px auto 40px;
  padding: 0 18px 32px;
}

.hero {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 24px 28px;
  border-radius: 24px;
  background: linear-gradient(135deg, #e0f2fe, #f8fbff);
  border: 1px solid rgba(14, 165, 233, 0.2);
}

.hero-text h1 {
  margin: 10px 0;
  font-size: 26px;
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

.layout-grid {
  display: grid;
  grid-template-columns: minmax(0, 2.1fr) minmax(0, 1.1fr);
  gap: 20px;
  margin-top: 20px;
}

.schedule-card,
.manage-card {
  padding: 22px;
}

.schedule-meta {
  display: flex;
  gap: 20px;
  margin-top: 10px;
}

.label {
  color: #94a3b8;
  margin: 0;
}

.value {
  margin: 2px 0 0;
  font-weight: 700;
}

.muted {
  color: #6b7280;
  margin: 8px 0 16px;
  font-size: 14px;
}

.muted.small {
  margin-top: 4px;
  font-size: 12px;
}

.today-session {
  padding: 10px 12px;
  border-radius: 14px;
  border: 1px dashed rgba(148, 163, 184, 0.7);
  background: #f8fafc;
  margin-bottom: 14px;
}

.table-shell {
  margin-top: 16px;
  border-radius: 18px;
  border: 1px solid #e5e7eb;
  background: #ffffff;
  box-shadow: 0 10px 26px rgba(15, 23, 42, 0.05);
  padding: 10px 16px 12px;
  max-height: 340px;
  overflow-y: auto;
}

.table-header-row,
.table-row {
  display: grid;
  grid-template-columns: 50px 1.7fr 1fr 1fr;
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

.empty-copy {
  margin: 4px 0 12px;
  color: #6b7280;
  font-size: 14px;
}

.pill-btn {
  border-radius: 999px;
  font-weight: 600;
}

@media (max-width: 768px) {
  .hero {
    flex-direction: column;
    align-items: flex-start;
    gap: 16px;
  }

  .layout-grid {
    grid-template-columns: 1fr;
  }

  .table-header-row,
  .table-row {
    grid-template-columns: 40px 1.4fr 0.9fr 0.9fr;
  }
}
</style>


