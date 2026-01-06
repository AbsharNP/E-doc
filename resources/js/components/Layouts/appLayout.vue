<template>
  <div class="app-layout">
    <NavBar />

    <div class="main-content">
      <router-view></router-view>
    </div>

    <footer class="app-footer">
      <div class="footer-inner">
        <div>
          <h6>Edoc Care</h6>
          <p>Modern, secure and patient-first healthcare platform.</p>
        </div>
        <div class="footer-meta">
          <span>© 2024 Edoc. All rights reserved.</span>
          <span class="dot"></span>
          <span>HIPAA-ready | Encrypted</span>
        </div>
      </div>
    </footer>
  </div>
</template>

<script>
import NavBar from '../layoutComponents/NavBar.vue';

export default {
  name: 'AppLayout',
  components: {
    NavBar,
  },
};
</script>

<style scoped>
html, body {
  height: 100%; /* Ensure full height for the body */
  margin: 0; /* Remove default margins */
}

.app-layout {
  display: flex;
  flex-direction: column;
  min-height: 100vh; /* Full viewport height */
  background: linear-gradient(180deg, #f8fbff 0%, #f5f7fb 60%, #eef2f7 100%);
}

.main-content {
  margin-top: 70px; /* Adjust this value based on your navbar height */
  flex: 1; /* Allow this section to grow and fill the available space */
}

.app-footer {
  padding: 20px 28px 28px;
  background: rgba(255, 255, 255, 0.9);
  border-top: 1px solid rgba(229, 231, 235, 0.7);
  box-shadow: 0 -8px 30px rgba(14, 165, 233, 0.05);
}

.footer-inner {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 1200px;
  margin: 0 auto;
  gap: 18px;
  color: #475569;
}

.footer-meta {
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 600;
}

.dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #94a3b8;
}

@media (max-width: 768px) {
  .footer-inner {
    flex-direction: column;
    text-align: center;
  }
}
</style>
