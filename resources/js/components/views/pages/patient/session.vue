<template>
  <div class="session-page">
    <div class="hero glass-panel" v-if="doctor">
      <div class="hero-text">
        <div class="pill">Choose a time</div>
        <h1>Pick a slot with {{ doctor.name }}</h1>
        <p>
          Select the day and an exact time slot that works for you.
          We’ll reserve that slot for your appointment.
        </p>
      </div>
    </div>

    <div v-if="error" class="alert alert-danger text-center mb-3">
      {{ error }}
    </div>

    <div v-if="!filteredSessions.length && !error" class="alert alert-info text-center">
      <p>No available sessions at this time. Please check back later.</p>
    </div>

    <div v-if="filteredSessions.length" class="cards-grid">
      <div
        v-for="session in filteredSessions"
        :key="session.id"
        class="card-elevated session-card"
      >
        <div class="card-header">
          <div>
            <p class="label">Date</p>
            <p class="value">{{ formatDate(session.session_date) }}</p>
          </div>
          <div>
            <p class="label">Window</p>
            <p class="value">
              {{ formatTime(session.start_time) }} – {{ formatTime(session.end_time) }}
            </p>
          </div>
          <div>
            <p class="label">Slots</p>
            <p class="value">{{ timeSlots(session).length }}</p>
          </div>
        </div>
        <div class="slots">
          <p class="slots-label">Available time slots</p>
          <div class="slots-grid">
            <button
              v-for="slot in timeSlots(session)"
              :key="slot.raw"
              type="button"
              :class="[
                'slot-pill',
                selectedSlot[session.id] === slot.raw ? 'active' : ''
              ]"
              @click="selectSlot(session.id, slot.raw)"
            >
              {{ slot.label }}
            </button>
          </div>
        </div>
        <div class="card-actions">
          <button
            class="btn btn-primary btn-sm"
            :disabled="!selectedSlot[session.id]"
            @click="submitBooking(session.id)"
          >
            Book selected slot
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import { useRoute } from 'vue-router';
import axios from 'axios';
import Swal from 'sweetalert2';

export default {
  setup() {
    const route = useRoute();
    const sessions = ref([]);
    const error = ref(null);
    const doctor = ref(null);
    const doctorId = ref(route.query.doctorId);
    const selectedSlot = ref({});

    onMounted(() => {
      fetchSessions();
    });

    const fetchSessions = async () => {
      try {
        const response = await axios.post(`/sessions/${doctorId.value}`);
        const { doctor: doctorData, sessions: allSessions } = response.data;

        allSessions.sort((a, b) => new Date(a.session_date) - new Date(b.session_date));

        doctor.value = doctorData;
        sessions.value = allSessions;
      } catch (err) {
        error.value = 'Failed to load sessions. Please try again.';
      }
    };

    const filteredSessions = computed(() => {
      const today = new Date();
      today.setHours(0, 0, 0, 0);

      return sessions.value.filter((session) => {
        const sessionDate = new Date(session.session_date);
        return sessionDate >= today;
      });
    });

    const formatDate = (date) => new Date(date).toLocaleDateString();
    const formatTime = (time) => {
      const [hours, minutes] = time.split(':');
      const date = new Date();
      date.setHours(hours, minutes);
      return date.toLocaleTimeString([], {
        hour: '2-digit',
        minute: '2-digit',
        hour12: true,
      });
    };

    // Generate 20-minute slots between start and end time
    const timeSlots = (session) => {
      const slots = [];
      const [startHour, startMin] = session.start_time.split(':').map(Number);
      const [endHour, endMin] = session.end_time.split(':').map(Number);

      const start = new Date();
      start.setHours(startHour, startMin, 0, 0);
      const end = new Date();
      end.setHours(endHour, endMin, 0, 0);

      const stepMinutes = 20;

      const cursor = new Date(start);
      while (cursor < end) {
        const raw =
          String(cursor.getHours()).padStart(2, '0') +
          ':' +
          String(cursor.getMinutes()).padStart(2, '0');

        slots.push({
          raw,
          label: cursor.toLocaleTimeString([], {
            hour: '2-digit',
            minute: '2-digit',
            hour12: true,
          }),
        });

        cursor.setMinutes(cursor.getMinutes() + stepMinutes);
      }

      return slots;
    };

    const selectSlot = (sessionId, slotRaw) => {
      selectedSlot.value = {
        ...selectedSlot.value,
        [sessionId]: slotRaw,
      };
    };

    const submitBooking = async (sessionId) => {
      const userId = localStorage.getItem('userId');
      const userName = localStorage.getItem('userName');
      const slot = selectedSlot.value[sessionId];

      if (!doctor.value) {
        Swal.fire('Error', 'Doctor information is not available', 'error');
        return;
      }

      if (!slot) {
        Swal.fire('Select a slot', 'Please pick a time slot before booking.', 'info');
        return;
      }

      try {
        const response = await axios.post('/appointments', {
          user_id: userId,
          name: userName,
          session_id: sessionId,
          dr_name: doctor.value.name,
          doctor_id: doctorId.value,
          slot_time: slot,
        });

        Swal.fire('Success', 'Appointment booked successfully', 'success');
      } catch (error) {
        Swal.fire('Error', error.response.data.message || 'Failed to book appointment', 'error');
      }
    };

    return {
      sessions,
      error,
      doctor,
      filteredSessions,
      formatDate,
      formatTime,
      timeSlots,
      selectedSlot,
      selectSlot,
      submitBooking,
    };
  }
};
</script>

<style scoped>
.session-page {
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

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 18px;
}

.session-card {
  padding: 18px 18px 14px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  gap: 10px;
}

.label {
  color: #94a3b8;
  margin: 0;
  font-size: 12px;
}

.value {
  margin: 2px 0 0;
  font-weight: 600;
  font-size: 14px;
}

.slots {
  margin-top: 8px;
}

.slots-label {
  font-size: 13px;
  font-weight: 600;
  color: #64748b;
  margin-bottom: 6px;
}

.slots-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.slot-pill {
  border-radius: 999px;
  border: 1px solid #e5e7eb;
  background: #ffffff;
  padding: 6px 12px;
  font-size: 12px;
  color: #0f172a;
  cursor: pointer;
  transition: all 0.15s ease;
}

.slot-pill:hover {
  border-color: #0ea5e9;
  background: #f0f9ff;
}

.slot-pill.active {
  background: #0ea5e9;
  color: #ffffff;
  border-color: #0ea5e9;
  box-shadow: 0 8px 20px rgba(14, 165, 233, 0.25);
}

.card-actions {
  margin-top: 6px;
  display: flex;
  justify-content: flex-end;
}

@media (max-width: 768px) {
  .card-header {
    flex-direction: column;
  }
}
</style>
