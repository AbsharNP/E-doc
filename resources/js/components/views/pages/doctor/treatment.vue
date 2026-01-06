<template>
  <div class="prescription-page">
    <div class="hero glass-panel">
      <div class="hero-text">
        <div class="pill">Treatment</div>
        <h1>Prescription & plan</h1>
        <p>
          Review patient details, capture your prescription, and add treatments or scans
          as a structured plan for this visit.
        </p>
      </div>
    </div>

    <div class="layout-grid">
      <div class="card-elevated left-column">
        <div class="section-heading">Patient</div>
        <div class="patient-info">
          <p><span class="label">Name</span><span class="value">{{ appointment.name }}</span></p>
          <p>
            <span class="label">Appointment date</span>
            <span class="value">{{ formatDate(appointment.session_date) }}</span>
          </p>
          <div class="age-row">
            <span class="label">Age</span>
            <input
              type="number"
              v-model="age"
              min="0"
              class="form-control age-input"
              placeholder="Enter age"
              required
            />
          </div>
        </div>

        <div class="section-heading">Facility</div>
        <div class="facility-info">
          <p class="value"><strong>Address:</strong> {{ facilityAddress }}</p>
          <p class="value"><strong>Phone:</strong> {{ facilityPhone }}</p>
          <p class="value"><strong>Attending:</strong> Dr. {{ doctorName }}</p>
        </div>
      </div>

      <div class="card-elevated right-column">
        <div class="section-heading">Prescription</div>
        <textarea
          v-model="prescription"
          class="form-control rx-textarea"
          placeholder="Enter prescription details here"
          rows="5"
        ></textarea>

        <div class="section-heading mt-3">Treatments</div>
        <div class="multi-list">
          <div
            class="multi-item"
            v-for="(item, index) in treatments"
            :key="'t-' + index"
          >
            <input
              v-model="treatments[index]"
              type="text"
              class="form-control"
              :placeholder="`Treatment ${index + 1}`"
            />
          </div>
          <button
            type="button"
            class="btn btn-outline-primary btn-sm pill-btn mt-2"
            @click="addTreatmentField"
          >
            + Add treatment
          </button>
        </div>

        <div class="section-heading mt-3">Scans / investigations</div>
        <div class="multi-list">
          <div
            class="multi-item"
            v-for="(item, index) in scans"
            :key="'s-' + index"
          >
            <input
              v-model="scans[index]"
              type="text"
              class="form-control"
              :placeholder="`Scan / investigation ${index + 1}`"
            />
          </div>
          <button
            type="button"
            class="btn btn-outline-primary btn-sm pill-btn mt-2"
            @click="addScanField"
          >
            + Add scan / investigation
          </button>
        </div>

        <div class="submit-section">
          <button class="btn btn-primary pill-btn" @click="confirmSubmission">
            Submit prescription
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
  
<script>
import axios from 'axios';
import Swal from 'sweetalert2'; // Import SweetAlert2

export default {
    data() {
        return {
            appointment: {
                name: '',
                phone_number: '',
                session_date: '', // Note: This will be formatted in the display
            },
            doctorId: localStorage.getItem('doctorId'),
            prescription: '',  
            age:'',
            treatments: [''],
            scans: [''],
            facilityAddress: '123 Clinic Street, City Name',
            facilityPhone: '+1-234-567-890',
            doctorName: localStorage.getItem('userName'), // Attending physician
            doctorPhone: '+1-987-654-321',
        };
    },
    mounted() {
        this.getAppointmentDetails();
    },
    methods: {
        getAppointmentDetails() {
            const appointmentId = this.$route.params.id; // Retrieve the ID from the route parameters
            console.log('Appointment ID:', appointmentId); // Log to check the ID

            if (!appointmentId) {
                console.error("Appointment ID is undefined."); // Handle undefined ID case
                return; // Exit if ID is undefined
            }

            // Make the API call to fetch the appointment details
            axios.post(`/treatment/${appointmentId}`)
                .then(response => {
                    console.log('Response from API:', response);
                    this.appointment = response.data; // Assign the response data
                    console.log('Appointment Details:', this.appointment); // Log details for debugging
                })
                .catch(error => {
                    console.error("Error fetching appointment:", error.response ? error.response.data : error); // Log error details
                });
        },
        confirmSubmission() {
            Swal.fire({
                title: 'Are you sure?',
                text: "Do you want to submit the prescription?",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Yes, submit it!'
            }).then((result) => {
                if (result.isConfirmed) {
                    this.submitPrescription(); // Call the prescription submission method
                }
            });
        },
        submitPrescription() {
            const treatmentLines = this.treatments
                .filter((t) => t && t.trim().length > 0)
                .map((t) => `- ${t.trim()}`)
                .join('\n');

            const scanLines = this.scans
                .filter((s) => s && s.trim().length > 0)
                .map((s) => `- ${s.trim()}`)
                .join('\n');

            let composedPrescription = this.prescription || '';

            if (treatmentLines) {
                composedPrescription += `\n\nTreatments:\n${treatmentLines}`;
            }
            if (scanLines) {
                composedPrescription += `\n\nScans / Investigations:\n${scanLines}`;
            }

            const prescriptionData = {
                appointmentId: this.$route.params.id,
                prescription: composedPrescription,
                age: this.age,
                treatment_status: 1, // Assuming treatment status 1 means completed
            };

            axios.post('/submit-prescription', prescriptionData) // Update the URL for consistency
                .then(response => {
                    Swal.fire(
                        'Submitted!',
                        'Prescription submitted successfully!',
                        'success'
                    );
                    this.$router.push('/appointment');  // Navigate back to the appointments list
                })
                .catch(error => {
                    console.error("Error submitting prescription:", error.response ? error.response.data : error);
                    Swal.fire({
                        icon: 'error',
                        title: 'Oops...',
                        text: 'Failed to submit the prescription. Please try again.',
                    });
                });
        },
        // Format date
        formatDate(dateString) {
            const options = { day: '2-digit', month: '2-digit', year: '2-digit' };
            const date = new Date(dateString);
            return date.toLocaleDateString('en-GB', options);  // Format date as 'dd/mm/yyyy'
        },
        addTreatmentField() {
            this.treatments.push('');
        },
        addScanField() {
            this.scans.push('');
        },
    },
};
</script>

<style scoped>
.prescription-page {
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
  font-size: 24px;
}

.hero-text p {
  color: #475569;
  max-width: 540px;
}

.layout-grid {
  display: grid;
  grid-template-columns: minmax(0, 1.1fr) minmax(0, 1.6fr);
  gap: 18px;
}

.left-column,
.right-column {
  padding: 18px 18px 16px;
}

.patient-info p {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  margin: 4px 0;
}

.label {
  color: #94a3b8;
  font-size: 13px;
}

.value {
  font-weight: 600;
  color: #0f172a;
}

.age-row {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 6px;
}

.age-input {
  max-width: 120px;
}

.facility-info {
  margin-top: 8px;
}

.facility-info .value {
  display: block;
  margin-bottom: 4px;
}

.rx-textarea {
  resize: vertical;
}

.multi-list {
  margin-top: 6px;
}

.multi-item + .multi-item {
  margin-top: 6px;
}

.submit-section {
  margin-top: 18px;
  display: flex;
  justify-content: flex-end;
}

.pill-btn {
  border-radius: 999px;
  font-weight: 600;
}

@media (max-width: 768px) {
  .layout-grid {
    grid-template-columns: 1fr;
  }
}
</style>
