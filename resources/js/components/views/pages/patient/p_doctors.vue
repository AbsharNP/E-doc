<template>
  <div class="doctor-list-page">
    <!-- Hero -->
    <div class="hero glass-panel">
      <div class="hero-text">
        <div class="pill">Book a visit</div>
        <h1>Find the right doctor for you</h1>
        <p>
          Search by doctor name or quickly filter by department,
          then jump into available sessions to confirm your booking.
        </p>
      </div>
    </div>

    <!-- Filters -->
    <div class="filters card-elevated">
      <div class="filters-row">
        <div class="filter-group">
          <label class="filter-label">Doctor name</label>
          <input
            v-model="searchQuery"
            type="text"
            class="form-control"
            placeholder="Search by doctor name"
          />
        </div>
        <div class="filter-group">
          <label class="filter-label">Department</label>
          <select v-model="selectedDepartment" class="form-select">
            <option value="all">All departments</option>
            <option
              v-for="dep in departments"
              :key="dep"
              :value="dep"
            >
              {{ dep }}
            </option>
          </select>
        </div>
      </div>
    </div>

    <!-- Doctors grid -->
    <div class="cards-grid">
      <div
        class="card-elevated doctor-card"
        v-for="doctor in filteredDoctors"
        :key="doctor.id"
      >
        <div class="card-header">
          <div>
            <h3>{{ doctor.name }}</h3>
            <p class="dept">
              {{ doctor.department ? doctor.department.name : 'No department' }}
            </p>
          </div>
        </div>
        <div class="card-body">
          <div class="meta">
            <p><span class="label">Doctor ID</span><span class="value">{{ doctor.doc_id }}</span></p>
            <p><span class="label">Email</span><span class="value">{{ doctor.email }}</span></p>
            <p><span class="label">Phone</span><span class="value">{{ doctor.phone_number }}</span></p>
          </div>
        </div>
        <div class="card-actions">
          <button class="btn btn-primary pill-btn" @click="getAppointment(doctor.id)">
            View available sessions
          </button>
        </div>
      </div>

      <div v-if="filteredDoctors.length === 0" class="empty-state">
        <p>No doctors match your filters. Try adjusting the name or department.</p>
      </div>
    </div>

    <div v-if="error" class="alert alert-danger mt-4 text-center">{{ error }}</div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';

export default {
  setup() {
    const router = useRouter();
    const doctors = ref([]);
    const searchQuery = ref('');
    const selectedDepartment = ref('all');
    const error = ref(null);

    onMounted(() => {
      fetchDoctors();
    });

    const fetchDoctors = async () => {
      try {
        const response = await axios.post('/doctor');
        doctors.value = response.data;
      } catch (err) {
        console.error('Error fetching doctors:', err);
        error.value = 'Failed to load doctors. Please try again.';
      }
    };

    const departments = computed(() => {
      const set = new Set();
      doctors.value.forEach((d) => {
        if (d.department && d.department.name) {
          set.add(d.department.name);
        }
      });
      return Array.from(set).sort();
    });

    const filteredDoctors = computed(() => {
      const nameTerm = searchQuery.value.trim().toLowerCase();
      const dep = selectedDepartment.value;

      return doctors.value.filter((doctor) => {
        const matchesName =
          !nameTerm ||
          (doctor.name && doctor.name.toLowerCase().includes(nameTerm));

        const doctorDep = doctor.department ? doctor.department.name : null;
        const matchesDepartment =
          dep === 'all' || (doctorDep && doctorDep === dep);

        return matchesName && matchesDepartment;
      });
    });

    const getAppointment = async (doctorId) => {
      try {
        await axios.get(`/sessions/${doctorId}`);
        router.push({
          path: '/available-sessions',
          query: { doctorId: doctorId.toString() },
        });
      } catch (err) {
        console.error('Error fetching sessions:', err);
        error.value = 'Failed to load sessions. Please try again.';
      }
    };

    return {
      doctors,
      searchQuery,
      selectedDepartment,
      departments,
      filteredDoctors,
      error,
      getAppointment,
    };
  },
};
</script>

<style scoped>
.doctor-list-page {
  max-width: 1200px;
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
  font-size: 26px;
}

.hero-text p {
  color: #475569;
  max-width: 540px;
}

.filters {
  padding: 16px 18px;
  margin-bottom: 20px;
}

.filters-row {
  display: grid;
  grid-template-columns: minmax(0, 2fr) minmax(0, 1.4fr);
  gap: 14px;
}

.filter-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.filter-label {
  font-size: 13px;
  font-weight: 600;
  color: #64748b;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 18px;
  margin-top: 10px;
}

.doctor-card {
  padding: 18px 18px 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.card-header h3 {
  margin: 0;
  font-size: 18px;
}

.dept {
  margin: 4px 0 0;
  color: #0ea5e9;
  font-weight: 600;
  font-size: 13px;
}

.meta p {
  display: flex;
  justify-content: space-between;
  gap: 8px;
  margin: 4px 0;
  font-size: 13px;
}

.label {
  color: #94a3b8;
}

.value {
  font-weight: 600;
  color: #0f172a;
}

.card-actions {
  margin-top: 8px;
  display: flex;
  justify-content: flex-end;
}

.empty-state {
  text-align: center;
  color: #94a3b8;
  grid-column: 1 / -1;
  padding: 16px 0;
}

.pill-btn {
  border-radius: 999px;
  padding-inline: 16px;
  font-weight: 600;
}

@media (max-width: 768px) {
  .filters-row {
    grid-template-columns: 1fr;
  }
}
</style>
