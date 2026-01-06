<template>
  <div class="admin-page">
    <!-- Hero / header -->
    <div class="hero glass-panel">
      <div class="hero-text">
        <div class="pill">Admin control</div>
        <h1>Operational health at a glance</h1>
        <p>
          Track appointment performance, manage departments and doctors,
          and keep your medical network in sync.
        </p>
      </div>
    </div>

    <!-- Analytics row -->
    <div class="analytics-row">
      <div class="card-elevated stat-card">
        <p class="label">Departments</p>
        <p class="stat">{{ departments.length }}</p>
      </div>
      <div class="card-elevated stat-card">
        <p class="label">Doctors</p>
        <p class="stat">{{ doctors.length }}</p>
      </div>
      <div class="card-elevated stat-card">
        <p class="label">Appointments</p>
        <p class="stat">{{ totalAppointments }}</p>
      </div>
    </div>

    <!-- Main layout -->
    <div class="layout-grid">
      <!-- Departments card -->
      <div class="card-elevated table-card">
        <div class="section-heading-row">
          <div class="section-heading">Departments</div>
          <button class="btn btn-outline-primary btn-sm pill-btn" @click="showAddDepartmentModal">
            Add department
          </button>
        </div>
        <div class="table-shell">
          <div class="table-header-row">
            <span>#</span>
            <span>Name</span>
            <span class="text-end">Actions</span>
          </div>
          <div
            class="table-row"
            v-for="(department, index) in departments"
            :key="department.id"
          >
            <span>{{ index + 1 }}</span>
            <span>{{ department.name }}</span>
            <span class="text-end">
              <div class="action-group">
                <button
                  class="btn btn-outline-danger btn-sm pill-btn"
                  @click="confirmDeleteDepartment(department.id)"
                >
                  Delete
                </button>
              </div>
            </span>
          </div>
        </div>
      </div>

      <!-- Doctors card -->
      <div class="card-elevated table-card">
        <div class="section-heading-row">
          <div class="section-heading">Doctors</div>
          <div class="table-header-actions">
            <router-link to="/add-doctor" class="btn btn-outline-primary btn-sm pill-btn">
              Add doctor
            </router-link>
          </div>
        </div>
        <div class="table-shell">
          <div class="table-header-row">
            <span>Name</span>
            <span>Email</span>
            <span>Department</span>
            <span class="text-end">Actions</span>
          </div>
          <div
            class="table-row"
            v-for="doctor in doctors"
            :key="doctor.id"
          >
            <span>{{ doctor.name }}</span>
            <span>{{ doctor.email }}</span>
            <span>{{ doctor.department ? doctor.department.name : 'N/A' }}</span>
            <span class="text-end">
              <div class="action-group">
                <button class="btn btn-outline-primary btn-sm pill-btn" @click="showDoctorDetails(doctor)">
                  View
                </button>
                <button class="btn btn-outline-danger btn-sm pill-btn" @click="confirmDeleteDoctor(doctor.id)">
                  Delete
                </button>
              </div>
            </span>
          </div>
          <div v-if="error" class="alert alert-danger mt-2">{{ error }}</div>
        </div>
      </div>
    </div>

    <!-- Appointments analytics -->
    <div class="card-elevated analytics-card">
      <div class="section-heading">Appointments overview</div>
      <div class="chart-wrapper">
        <apexchart
          width="380"
          type="donut"
          :options="options"
          :series="series"
        ></apexchart>
      </div>
    </div>

    <!-- Modal for Adding Department -->
    <div
      class="modal fade"
      :class="{ show: isModalVisible }"
      :style="{ display: isModalVisible ? 'block' : 'none' }"
      tabindex="-1"
      role="dialog"
      @click.self="closeAddDepartmentModal"
    >
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header bg-primary text-white">
            <h5 class="modal-title">Add Department</h5>
          </div>
          <div class="modal-body">
            <form @submit.prevent="addDepartment">
              <div class="form-group">
                <label for="departmentName">Department Name</label>
                <input
                  type="text"
                  id="departmentName"
                  v-model="depName"
                  class="form-control"
                  required
                />
              </div>
              <div class="text-center mt-3 modal-actions">
                <button type="submit" class="btn btn-outline-primary pill-btn">Add</button>
                <span class="pd"></span>
                <button
                  type="button"
                  class="btn btn-outline-danger pill-btn"
                  @click="closeAddDepartmentModal"
                >
                  Cancel
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal for Doctor Details -->
    <div
      class="modal fade"
      :class="{ show: isDoctorModalVisible }"
      :style="{ display: isDoctorModalVisible ? 'block' : 'none' }"
      tabindex="-1"
      role="dialog"
      @click.self="closeDoctorModal"
    >
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header bg-primary text-white">
            <h5 class="modal-title">Doctor Details</h5>
            <button type="button" class="close" @click="closeDoctorModal">
              <span>&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <p><strong>Name:</strong> {{ selectedDoctor.name }}</p>
            <p><strong>Doctor ID:</strong> {{ selectedDoctor.doc_id }}</p>
            <p><strong>Email:</strong> {{ selectedDoctor.email }}</p>
            <p><strong>Phone Number:</strong> {{ selectedDoctor.phone_number }}</p>
            <p>
              <strong>Department:</strong>
              {{ selectedDoctor.department ? selectedDoctor.department.name : 'N/A' }}
            </p>
          </div>
          <div class="modal-footer modal-actions">
            <button
              type="button"
              class="btn btn-outline-danger pill-btn"
              @click="closeDoctorModal"
            >
              Close
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Backdrop for Modal -->
    <div
      v-if="isModalVisible || isDoctorModalVisible"
      class="modal-backdrop fade show"
    ></div>
  </div>
</template>

<script>
import axios from 'axios';
import VueApexCharts from 'vue3-apexcharts';
import Swal from 'sweetalert2';

export default {
  name: 'AdminHomeModern',
  components: {
    apexchart: VueApexCharts,
  },
  data() {
    return {
      departments: [],
      doctors: [],
      isModalVisible: false,
      isDoctorModalVisible: false,
      selectedDoctor: {},
      depName: '',
      selectedDepartmentId: null,
      error: null,
      options: {
        chart: {
          id: 'appointments-chart',
        },
        labels: ['Completed', 'Not Completed'],
        colors: ['#22c55e', '#ef4444'],
        legend: {
          position: 'bottom',
        },
      },
      series: [],
    };
  },
  computed: {
    totalAppointments() {
      if (!this.series.length) return 0;
      return this.series.reduce((sum, v) => sum + v, 0);
    },
  },
  methods: {
    fetchAppointmentStats() {
      axios
        .post('/analyse')
        .then((response) => {
          const { completed, not_completed } = response.data;
          this.series = [completed, not_completed];
        })
        .catch((error) => {
          console.error('Error fetching appointment stats:', error);
        });
    },
    showAddDepartmentModal() {
      this.isModalVisible = true;
    },
    closeAddDepartmentModal() {
      this.isModalVisible = false;
      this.depName = '';
      this.selectedDepartmentId = null;
    },
    fetchDepartments() {
      axios
        .post('/departments')
        .then((response) => {
          this.departments = response.data;
        })
        .catch((error) => {
          console.error('Error fetching departments:', error);
        });
    },
    fetchDoctors() {
      axios
        .post('/doctors')
        .then((response) => {
          this.doctors = response.data;
        })
        .catch((error) => {
          console.error('Error fetching doctors:', error);
          this.error = 'An error occurred while fetching doctors.';
        });
    },
    addDepartment() {
      axios
        .post('/add-department', {
          name: this.depName,
        })
        .then(() => {
          Swal.fire('Success', 'Department added successfully.', 'success');
          this.fetchDepartments();
          this.closeAddDepartmentModal();
        })
        .catch((error) => {
          console.error('Error adding department:', error);
        });
    },
    confirmDeleteDepartment(id) {
      Swal.fire({
        title: 'Are you sure?',
        text: 'This will delete the department and all associated data.',
        icon: 'warning',
        showCancelButton: true,
        confirmButtonText: 'Yes, delete it!',
        cancelButtonText: 'Cancel',
      }).then((result) => {
        if (result.isConfirmed) {
          this.deleteDepartment(id);
        }
      });
    },
    deleteDepartment(id) {
      axios
        .delete(`/depdlt/${id}`)
        .then(() => {
          Swal.fire('Deleted', 'Department has been deleted.', 'success');
          this.fetchDepartments();
        })
        .catch((error) => {
          console.error('Error deleting department:', error);
        });
    },
    confirmDeleteDoctor(id) {
      Swal.fire({
        title: 'Are you sure?',
        text: 'This will delete the doctor and all associated data.',
        icon: 'warning',
        showCancelButton: true,
        confirmButtonText: 'Yes, delete it!',
        cancelButtonText: 'Cancel',
      }).then((result) => {
        if (result.isConfirmed) {
          this.deleteDoctor(id);
        }
      });
    },
    deleteDoctor(id) {
      axios
        .delete(`/doc/${id}`)
        .then(() => {
          Swal.fire('Deleted', 'Doctor has been deleted.', 'success');
          this.fetchDoctors();
        })
        .catch((error) => {
          console.error('Error deleting doctor:', error);
        });
    },
    showDoctorDetails(doctor) {
      this.selectedDoctor = doctor;
      this.isDoctorModalVisible = true;
    },
    closeDoctorModal() {
      this.isDoctorModalVisible = false;
      this.selectedDoctor = {};
    },
  },
  mounted() {
    this.fetchDepartments();
    this.fetchDoctors();
    this.fetchAppointmentStats();
  },
};
</script>

<style scoped>
.admin-page {
  max-width: 1200px;
  margin: 20px auto 40px;
  padding: 0 18px 32px;
}

.hero {
  padding: 22px 26px;
  border-radius: 24px;
  background: linear-gradient(135deg, #e0f2fe, #f8fbff);
  border: 1px solid rgba(14, 165, 233, 0.2);
  margin-bottom: 20px;
}

.hero-text h1 {
  margin: 6px 0 4px;
  font-size: 26px;
}

.hero-text p {
  color: #475569;
  max-width: 520px;
}

.analytics-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 14px;
  margin-bottom: 20px;
}

.stat-card {
  padding: 16px 18px;
}

.stat {
  font-size: 24px;
  font-weight: 700;
  color: #0f172a;
}

.layout-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(340px, 1fr));
  gap: 20px;
  margin-bottom: 20px;
}

.table-card {
  padding: 20px;
}

.section-heading-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  margin-bottom: 10px;
}

.table-header-actions {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 8px;
}

.table-shell {
  margin-top: 8px;
  border-radius: 18px;
  border: 1px solid #e5e7eb;
  background: #ffffff;
  box-shadow: 0 10px 26px rgba(15, 23, 42, 0.05);
  padding: 10px 16px 12px;
  max-height: 360px;
  overflow-y: auto;
}

.table-header-row,
.table-row {
  display: grid;
  gap: 10px;
  align-items: center;
  font-size: 14px;
}

.table-header-row {
  grid-template-columns: 50px 1.7fr 1fr;
  font-weight: 700;
  color: #64748b;
  border-bottom: 1px solid #e5e7eb;
  padding-bottom: 8px;
  margin-bottom: 4px;
}

.table-row {
  grid-template-columns: 50px 1.7fr 1fr;
  padding: 6px 0;
  border-radius: 10px;
}

.table-row:nth-child(odd) {
  background: #f8fafc;
}

.table-card:nth-child(2) .table-header-row,
.table-card:nth-child(2) .table-row {
  grid-template-columns: 1.3fr 1.4fr 1.2fr 1fr;
}

.action-group {
  display: inline-flex;
  gap: 6px;
  align-items: center;
  justify-content: flex-end;
}

.pill-btn {
  border-radius: 999px;
  padding-inline: 14px;
  font-weight: 600;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
}

.text-end {
  text-align: right;
}

.analytics-card {
  padding: 20px;
}

.chart-wrapper {
  display: flex;
  justify-content: center;
  padding-top: 10px;
}

.modal-backdrop {
  z-index: 1040;
  background-color: rgba(15, 23, 42, 0.4);
}

.modal {
  z-index: 1050;
}

@media (max-width: 768px) {
  .table-header-row,
  .table-row {
    font-size: 13px;
  }
}
</style>


