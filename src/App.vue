<script setup>
import { ref, watch } from 'vue'

const name = ref('')
const email = ref('')
const phone = ref('')
const password = ref('')
const confirmPassword = ref('')
const birthDate = ref('')
const country = ref('')
const file = ref(null)
const agree = ref(false)
const captchaInput = ref('')
const captchaCode = ref('')
const submitted = ref(false)
const imageUrl = ref('') // URL สำหรับแสดงภาพตัวอย่าง

const generateCaptcha = () => {
  const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
  let result = ''
  for (let i = 0; i < 6; i++) {
    result += characters.charAt(Math.floor(Math.random() * characters.length))
  }
  captchaCode.value = result
}

generateCaptcha()

const errors = ref({})

const validateEmail = (email) => {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return re.test(email)
}

const validatePhone = (phone) => {
  return /^[0-9]{10}$/.test(phone)
}

const validateFile = (file) => {
  if (!file) return false
  const validTypes = ['image/jpeg', 'image/png']
  const maxSize = 5 * 1024 * 1024
  return validTypes.includes(file.type) && file.size <= maxSize
}

const validateForm = () => {
  errors.value = {}

  if (!name.value) errors.value.name = "กรุณากรอกชื่อ"
  if (!email.value || !validateEmail(email.value)) errors.value.email = "กรุณากรอกอีเมลให้ถูกต้อง"
  if (!phone.value || !validatePhone(phone.value)) errors.value.phone = "กรุณากรอกหมายเลขโทรศัพท์ให้ถูกต้อง (10 หลัก)"
  if (!password.value || password.value.length < 8) errors.value.password = "รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร"
  if (password.value !== confirmPassword.value) errors.value.confirmPassword = "รหัสผ่านทั้งสองช่องต้องตรงกัน"
  if (!birthDate.value || new Date(birthDate.value) >= new Date()) errors.value.birthDate = "วันเกิดต้องไม่ใช่วันปัจจุบันหรืออนาคต"
  if (!country.value) errors.value.country = "กรุณาเลือกประเทศ"
  if (!validateFile(file.value)) errors.value.file = "กรุณาอัปโหลดรูปภาพที่เป็น JPG หรือ PNG และขนาดไม่เกิน 5MB"
  if (captchaInput.value !== captchaCode.value) errors.value.captcha = "CAPTCHA ไม่ถูกต้อง"
  if (!agree.value) errors.value.agree = "กรุณายอมรับเงื่อนไขการใช้งาน"

  return Object.keys(errors.value).length === 0
}

const submitForm = () => {
  if (validateForm()) {
    submitted.value = true
    alert('Form Submitted')
  } else {
    alert("โปรดตรวจสอบข้อมูลในฟอร์มอีกครั้ง")
  }
}

// Watch for changes in file and update imageUrl for preview
watch(file, (newFile) => {
  if (newFile) {
    const reader = new FileReader()
    reader.onload = (e) => {
      imageUrl.value = e.target.result
    }
    reader.readAsDataURL(newFile)
  }
})
</script>

<template>
  <form @submit.prevent="submitForm" class="form-container">
    <h2 class="form-title">V&V Form</h2>

    <div class="form-group">
      <label>ชื่อ*:</label>
      <input v-model="name" type="text" class="form-input" />
      <p v-if="errors.name" class="form-error">{{ errors.name }}</p>
    </div>

    <div class="form-group">
      <label>อีเมล*:</label>
      <input v-model="email" type="email" class="form-input" />
      <p v-if="errors.email" class="form-error">{{ errors.email }}</p>
    </div>

    <div class="form-group">
      <label>หมายเลขโทรศัพท์*:</label>
      <input v-model="phone" type="tel" maxlength="10" class="form-input" />
      <p v-if="errors.phone" class="form-error">{{ errors.phone }}</p>
    </div>

    <div class="form-group">
      <label>รหัสผ่าน*:</label>
      <input v-model="password" type="password" class="form-input" />
      <p v-if="errors.password" class="form-error">{{ errors.password }}</p>
    </div>

    <div class="form-group">
      <label>ยืนยันรหัสผ่าน*:</label>
      <input v-model="confirmPassword" type="password" class="form-input" />
      <p v-if="errors.confirmPassword" class="form-error">{{ errors.confirmPassword }}</p>
    </div>

    <div class="form-group">
      <label>วันเกิด*:</label>
      <input v-model="birthDate" type="date" class="form-input" />
      <p v-if="errors.birthDate" class="form-error">{{ errors.birthDate }}</p>
    </div>

    <div class="form-group">
      <label>ประเทศ*:</label>
      <select v-model="country" class="form-select">
        <option disabled value="">กรุณาเลือกประเทศ</option>
        <option>Thailand</option>
        <option>USA</option>
        <option>Other</option>
      </select>
      <p v-if="errors.country" class="form-error">{{ errors.country }}</p>
    </div>

    <div class="form-group">
      <label>อัปโหลดรูปภาพ*:</label>
      <input @change="file = $event.target.files[0]" type="file" accept=".jpg,.png" class="form-input" />
      <p v-if="errors.file" class="form-error">{{ errors.file }}</p>
      <!-- Image Preview -->
      <div v-if="file">
        <img :src="imageUrl" alt="Image preview" class="image-preview"/>
      </div>
    </div>

    <div class="form-group">
      <label>CAPTCHA: (กรุณากรอกรหัสต่อไปนี้)</label>
      <p>{{ captchaCode }}</p>
      <input v-model="captchaInput" type="text" class="form-input" />
      <p v-if="errors.captcha" class="form-error">{{ errors.captcha }}</p>
    </div>

    <div class="form-group-checkbox">
      <input v-model="agree" type="checkbox" class="form-checkbox" />
      <label class="text-checkbox">ยอมรับเงื่อนไขการใช้งาน*</label>
      <p v-if="errors.agree" class="form-error">{{ errors.agree }}</p>
    </div>

    <button type="submit" class="submit-button">ส่งข้อมูล</button>
  </form>

  <!-- การ์ดแสดงข้อมูล -->
  <div v-if="submitted" class="card">
    <h3>ข้อมูลที่คุณกรอก</h3>
    <p><strong>ชื่อ:</strong> {{ name }}</p>
    <p><strong>อีเมล:</strong> {{ email }}</p>
    <p><strong>หมายเลขโทรศัพท์:</strong> {{ phone }}</p>
    <p><strong>วันเกิด:</strong> {{ birthDate }}</p>
    <p><strong>ประเทศ:</strong> {{ country }}</p>
    <p><strong>ยอมรับเงื่อนไข:</strong> {{ agree ? 'Yes' : 'No' }}</p>
    <p><strong>CAPTCHA ที่กรอก:</strong> {{ captchaInput }}</p>
    <!-- Display Image in Card -->
    <div v-if="file">
      <img :src="imageUrl" alt="Uploaded image" class="card-image"/>
    </div>
  </div>
</template>

<style scoped>
.form-error {
  color: red;
  font-size: 0.8rem;
}

.form-container {
  max-width: 500px;
  margin: 0 auto;
  padding: 2rem;
  background-color: #131111;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(255, 255, 255, 0.643);
}

.form-title {
  text-align: center;
  margin-bottom: 1.5rem;
  color: #ffffff;
  font-size: 1.5rem;
}

.form-group {
  margin-bottom: 0.5rem;
}

.form-input, .form-select {
  width: 100%;
  padding: 0.8rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 5px;
  transition: border-color 0.3s;
}

.form-input:focus, .form-select:focus {
  border-color: #66afe9;
  outline: none;
  box-shadow: 0 0 5px rgba(102, 175, 233, 0.6);
}

.form-checkbox {
  margin-right: 0.5rem;
}

.submit-button {
  width: 100%;
  padding: 0.8rem;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.3s;
}

.submit-button:hover {
  background-color: #45a049;
}

.text-checkbox {
  color: #ffffff;
}

/* เพิ่มระยะห่างระหว่างฟอร์มและการ์ด */
.card {
  margin-top: 2rem; /* เพิ่มระยะห่างที่นี่ */
  padding: 1rem;
  background-color: #000000;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(251, 250, 250, 0.642);
  color: #ffffff;
}

.card h3 {
  margin-bottom: 1rem;
  text-align: center;
  font-size: 2rem;
}

.card p {
  margin: 0.5rem 0;
}

.image-preview {
  max-width: 100%;
  height: auto;
  margin-top: 1rem;
}

.card-image {
  max-width: 100%;
  height: auto;
  margin-top: 1rem;
}
</style>
