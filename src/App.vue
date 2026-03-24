<script setup>
import { ref, onMounted } from 'vue'

const products = ref([])
const loading = ref(true)
const error = ref(null)

const n8n_url = 'http://localhost:5678/webhook-test/get-products'

const fetchProducts = async () => {
  loading.value = true
  error.value = null
  try {
    if (n8n_url === 'ใส่_URL_WEBHOOK_ของ_n8n_ที่นี่' || !n8n_url) {
      throw new Error('กรุณาใส่ n8n Webhook URL ในไฟล์ App.vue')
    }

    const response = await fetch(n8n_url)
    if (!response.ok) throw new Error(`ไม่สามารถโหลดข้อมูลได้ (HTTP ${response.status})`)
    
    const data = await response.json()
    
    // จัดการรูปแบบข้อมูลที่หลากหลายจาก n8n
    if (Array.isArray(data)) {
      products.value = data
    } else if (data.data && Array.isArray(data.data)) {
      products.value = data.data
    } else {
      products.value = Object.values(data)
    }
    
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchProducts()
})
</script>

<template>
  <div class="container">
    <header>
      <h1>ข้อมูลสินค้า</h1>
      <p class="subtitle">ดึงข้อมูลแบบ Real-time จาก Google Sheets ผ่าน n8n</p>
    </header>

    <div v-if="loading" class="loading-state">
      <div class="spinner"></div>
      <p>กำลังดึงข้อมูลล่าสุด...</p>
    </div>

    <div v-else-if="error" class="error-state">
      <div class="error-card">
        <p class="error-title">เกิดข้อผิดพลาด</p>
        <p>{{ error }}</p>
      </div>
      <button @click="fetchProducts" class="refresh-button">ลองใหม่อีกครั้ง</button>
    </div>

    <div v-else class="table-wrapper">
      <table>
        <thead>
          <tr>
            <th>รหัสสินค้า</th>
            <th>ชื่อสินค้า</th>
            <th style="text-align: right;">จำนวน</th>
            <th style="text-align: right;">ราคา (บาท)</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(product, index) in products" :key="index">
            <td>{{ product['รหัสสินค้า'] || product['id'] || product['ID'] || `P00${index + 1}` }}</td>
            <td style="font-weight: 500;">{{ product['ชื่อสินค้า'] }}</td>
            <td style="text-align: right; color: var(--text-muted);">{{ product['จำนวน'] }}</td>
            <td style="text-align: right;" class="price">฿{{ product['ราคา'] }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
