<script setup>
import { ref } from 'vue';

// State variables
const username = ref('');
const password = ref('');
const showPassword = ref(false);
const loginError = ref('');

const registeredUsers = [
  { username: 'staff1', password: 'password123', name: 'John Doe' },
  { username: 'staff2', password: 'password123', name: 'Jane Smith' },
  { username: 'kolektix', password: 'password123', name: 'Kolektix Scanner Staff' }
];

const togglePassword = () => {
  showPassword.value = !showPassword.value;
};

const emit = defineEmits(['login-success']);

const handleLogin = () => {
  loginError.value = '';
  const user = registeredUsers.find(
    u => u.username.toLowerCase() === username.value.toLowerCase().trim() && 
         u.password === password.value
  );

  if (user) {
    emit('login-success', user.name);
  } else {
    loginError.value = 'Username atau password salah / belum terdaftar dari kreator!';
  }
};
</script>

<template>
  <div class="mobile-wrapper">
    <!-- Header Section -->
    <div class="header-section">
      <div class="logo-container">
        <img src="/media/newkolektix.d744083c.gif" alt="Kolektix Logo" class="logo-gif" />
      </div>
    </div>

    <!-- Login Card Form -->
    <div class="login-card">
      <!-- Username/Password Login View -->
      <div class="card-content-wrapper">
        <h1 class="welcome-title">Staff Scanner Login</h1>
        <p class="welcome-subtitle">Masukkan username dan password yang didaftarkan oleh Kreator Anda.</p>

        <!-- Simple Alert Text if not registered or incorrect -->
        <div v-if="loginError" class="alert-box">
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="alert-icon">
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v3.75m9-.75a9 9 0 1 1-18 0 9 9 0 0 1 18 0Zm-9 3.75h.008v.008H12v-.008Z" />
          </svg>
          <span>{{ loginError }}</span>
        </div>

        <form @submit.prevent="handleLogin" class="form-container">
          <!-- Username Field -->
          <div class="input-group">
            <label class="input-label">Username</label>
            <div class="input-wrapper">
              <input 
                type="text" 
                v-model="username" 
                placeholder="Masukkan username" 
                class="form-input"
                required
              />
              <span class="input-icon-right">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="icon-svg-filled">
                  <path fill-rule="evenodd" d="M7.5 6a4.5 4.5 0 1 1 9 0 4.5 4.5 0 0 1-9 0ZM3.751 20.105a8.25 8.25 0 0 1 16.498 0 .75.75 0 0 1-.437.695A18.683 18.683 0 0 1 12 22.5c-2.786 0-5.433-.608-7.812-1.7a.75.75 0 0 1-.437-.695Z" clip-rule="evenodd" />
                </svg>
              </span>
            </div>
          </div>

          <!-- Password Field -->
          <div class="input-group">
            <label class="input-label">Password</label>
            <div class="input-wrapper">
              <input 
                :type="showPassword ? 'text' : 'password'" 
                v-model="password" 
                placeholder="Masukkan password" 
                class="form-input"
                required
              />
              <button type="button" @click="togglePassword" class="password-toggle-right">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="icon-svg-filled">
                  <path d="M12 15a3 3 0 1 0 0-6 3 3 0 0 0 0 6Z" />
                  <path fill-rule="evenodd" d="M1.323 11.447C2.811 6.976 7.028 3.75 12.001 3.75c4.97 0 9.185 3.223 10.675 7.69.12.362.12.752 0 1.113-1.487 4.471-5.705 7.697-10.677 7.697-4.97 0-9.186-3.223-10.675-7.69a1.762 1.762 0 0 1 0-1.113ZM17.25 12a5.25 5.25 0 1 1-10.5 0 5.25 5.25 0 0 1 10.5 0Z" clip-rule="evenodd" />
                </svg>
              </button>
            </div>
          </div>

          <!-- Submit Button -->
          <button type="submit" class="submit-btn">
            <span>Lanjutkan</span>
          </button>
        </form>

        <div style="font-size: 11px; color: #888; text-align: center; margin-top: 10px;">
          Hint Akun: <b>staff1</b> / <b>password123</b>
        </div>
      </div>

      <!-- Footer Terms -->
      <div class="terms-footer">
        Dengan masuk, Anda menyetujui <br />
        <a href="#" class="terms-link">Syarat & Ketentuan</a> Kolektix.
      </div>
    </div>
  </div>
</template>

<style scoped>
.mobile-wrapper {
  width: 100%;
  max-width: 100%;
  height: 100vh;
  height: 100dvh;
  background-color: var(--white);
  display: flex;
  flex-direction: column;
  position: relative;
  overflow: hidden;
}

@media (min-width: 480px) {
  .mobile-wrapper {
    max-width: 410px;
    height: 88vh;
    max-height: 840px;
    border-radius: 30px;
    box-shadow: 0 12px 30px rgba(0, 0, 0, 0.1);
    margin: auto;
  }
}

.header-section {
  background-color: var(--primary-base);
  padding: 32px 20px 24px 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  flex-shrink: 0;
}

.logo-container {
  margin-bottom: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.logo-gif {
  max-width: 130px;
  height: auto;
  object-fit: contain;
}

/* Login Card Content */
.login-card {
  flex-grow: 1;
  background-color: var(--white);
  border-top-left-radius: 16px;
  border-top-right-radius: 16px;
  margin-top: -16px;
  padding: 28px 24px 20px 24px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.welcome-title {
  font-family: var(--font-poppins);
  font-size: 20px;
  font-weight: 700;
  color: var(--dark);
  margin-bottom: 4px;
}

.welcome-subtitle {
  font-family: var(--font-poppins);
  font-size: 12px;
  color: var(--dark-grey);
  margin-bottom: 20px;
  line-height: 1.4;
}

/* Simple Alert Warning Block */
.alert-box {
  background-color: #fee2e2;
  border: 1px solid #fecaca;
  color: #b91c1c;
  border-radius: 8px;
  padding: 10px 14px;
  font-size: 12px;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 18px;
  line-height: 1.4;
}

.alert-icon {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
}

/* Form layouts */
.form-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-bottom: 16px;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.input-label {
  font-size: 13px;
  font-weight: 600;
  color: var(--dark);
  font-family: var(--font-sans);
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.input-icon-right, .password-toggle-right {
  position: absolute;
  right: 4px;
  color: var(--primary-base);
  display: flex;
  align-items: center;
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
}

.icon-svg-filled {
  width: 20px;
  height: 20px;
}

.form-input {
  width: 100%;
  padding: 8px 32px 8px 0;
  border: none;
  border-bottom: 1px solid var(--light-grey);
  border-radius: 0;
  font-size: 15px;
  font-family: var(--font-sans);
  color: var(--dark);
  background-color: transparent;
  outline: none;
  transition: border-color 0.2s;
}

.form-input.text-center {
  text-align: center;
  padding: 8px 0;
}

.form-input::placeholder {
  color: #b0b0b5;
}

.form-input:focus {
  border-color: var(--primary-base);
}

/* Buttons Styling */
.submit-btn {
  background-color: var(--primary-base);
  color: var(--white);
  border: none;
  padding: 12px 20px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.2s;
  font-family: var(--font-sans);
  margin-top: 8px;
}

.submit-btn:hover {
  background-color: var(--primary-light-700);
}

.btn-row {
  display: flex;
  gap: 12px;
  width: 100%;
  margin-top: 8px;
}

.back-btn {
  background-color: var(--white);
  color: var(--dark);
  border: 1px solid var(--light-grey);
  padding: 12px 20px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
  font-family: var(--font-sans);
}

.back-btn:hover {
  background-color: var(--primary-light);
}

.flex-1 {
  flex: 1;
}

/* Footer Section */
.terms-footer {
  margin-top: auto;
  padding-top: 24px;
  text-align: center;
  font-size: 11px;
  color: var(--dark-grey);
  line-height: 1.5;
  font-family: var(--font-sans);
}

.terms-link {
  color: var(--primary-base);
  text-decoration: none;
}

.terms-link:hover {
  text-decoration: underline;
}
</style>
