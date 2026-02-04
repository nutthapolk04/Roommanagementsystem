<script setup>
import { ref, onMounted } from 'vue';
import { db } from './firebase';
import { 
  collection, 
  onSnapshot, 
  addDoc, 
  updateDoc, 
  doc, 
  deleteDoc, 
  query, 
  orderBy,
  setDoc,
  getDoc
} from 'firebase/firestore';

import Sidebar from './components/Sidebar.vue';
import Dashboard from './views/Dashboard.vue';
import InvoiceForm from './views/InvoiceForm.vue';
import InvoiceHistory from './views/InvoiceHistory.vue';
import TenantManager from './views/TenantManager.vue';
import Settings from './views/Settings.vue';
import InvoicePreviewModal from './components/InvoicePreviewModal.vue';

// State
const activeTab = ref('dashboard');
const loading = ref(true);
const copySuccess = ref(false);

const ownerSettings = ref({
  apartmentName: 'พี่ปุ้ยห้องพัก',
  address: '30/21-22 ซ.สุขุมวิท 23 แขวงคลองเตยเหนือ เขตวัฒนา กรุงเทพฯ 10110',
  phoneNumber: '084-1956628',
  bankInfo: 'กสิกรไทย 063-2-46836-9',
  defaultElectricityRate: 7,
  defaultWaterRate: 100,
  defaultInternet: 0,
  cashPayment: true
});

const tenants = ref([]);
const invoices = ref([]);
const selectedInvoice = ref(null);

onMounted(async () => {
  // 1. Sync Settings
  const settingsDoc = doc(db, 'config', 'ownerSettings');
  onSnapshot(settingsDoc, (snapshot) => {
    if (snapshot.exists()) {
      ownerSettings.value = { ...ownerSettings.value, ...snapshot.data() };
    } else {
      // Initialize settings if they don't exist
      setDoc(settingsDoc, ownerSettings.value);
    }
  });

  // 2. Sync Tenants
  const tenantsCol = collection(db, 'tenants');
  onSnapshot(tenantsCol, (snapshot) => {
    tenants.value = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    loading.value = false;
  });

  // 3. Sync Invoices
  const invoicesCol = query(collection(db, 'invoices'), orderBy('createdAt', 'desc'));
  onSnapshot(invoicesCol, (snapshot) => {
    invoices.value = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
  });
});

const showToast = () => {
  copySuccess.value = true;
  setTimeout(() => copySuccess.value = false, 3000);
};

const handleSaveInvoice = async (invoice) => {
  try {
    const { id, ...invoiceData } = invoice; // Remove temporary ID
    await addDoc(collection(db, 'invoices'), {
        ...invoiceData,
        createdAt: Date.now()
    });
    activeTab.value = 'history';
    showToast();
  } catch (e) {
    console.error("Error adding invoice: ", e);
    alert("เกิดข้อผิดพลาดในการบันทึกบิล");
  }
};

const updateTenantMeter = async (tenantId, newMeter) => {
  try {
    const tenantRef = doc(db, 'tenants', tenantId);
    await updateDoc(tenantRef, {
      electricityPrev: newMeter
    });
  } catch (e) {
    console.error("Error updating meter: ", e);
  }
};

const handleUpdateTenants = async (newTenants) => {
    // This is for bulk update from TenantManager (mostly adds/edits)
    // Since we are using onSnapshot, our local tenants.value will update automatically 
    // when we push to Firestore.
    
    // Check for deleted (active = false) or new/updated
    for (const t of newTenants) {
        if (!t.id) {
            // New tenant
            const { id, ...data } = t;
            await addDoc(collection(db, 'tenants'), data);
        } else {
            // Update existing
            const tenantRef = doc(db, 'tenants', t.id);
            const { id, ...data } = t;
            await updateDoc(tenantRef, data);
        }
    }
};

const handleDeleteInvoice = async (invoiceId) => {
    if (confirm('ต้องการลบใบเสร็จนี้?')) {
        try {
            await deleteDoc(doc(db, 'invoices', invoiceId));
        } catch (e) {
            console.error("Error deleting invoice: ", e);
        }
    }
};

const handleUpdateSettings = async (newSettings) => {
    try {
        const settingsDoc = doc(db, 'config', 'ownerSettings');
        await updateDoc(settingsDoc, newSettings);
    } catch (e) {
        console.error("Error updating settings: ", e);
    }
};

</script>

<template>
  <div class="min-h-screen bg-slate-50 flex flex-col md:flex-row">
    <!-- Success Toast -->
    <Transition name="fade">
      <div v-if="copySuccess" class="fixed top-6 left-1/2 transform -translate-x-1/2 z-[200] bg-green-600 text-white px-6 py-3 rounded-2xl shadow-2xl flex items-center gap-2">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        <span class="font-bold">สำเร็จ!</span>
      </div>
    </Transition>

    <Sidebar v-model="activeTab" :apartmentName="ownerSettings.apartmentName" />

    <!-- Main Content -->
    <main class="flex-1 md:ml-64 p-6 pb-24 md:pb-8">
      <div v-if="loading" class="flex items-center justify-center h-full">
        <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-green-700"></div>
      </div>
      
      <div v-else>
        <Dashboard 
          v-if="activeTab === 'dashboard'" 
          :invoices="invoices" 
          :tenants="tenants" 
          :ownerSettings="ownerSettings"
          @navigate="activeTab = $event"
        />
        
        <InvoiceForm 
          v-if="activeTab === 'create'" 
          :tenants="tenants" 
          :ownerSettings="ownerSettings"
          @save="handleSaveInvoice"
          @update-meter="updateTenantMeter"
        />
        
        <InvoiceHistory 
          v-if="activeTab === 'history'" 
          :invoices="invoices"
          @view="selectedInvoice = $event"
          @delete="handleDeleteInvoice"
        />
        
        <TenantManager 
          v-if="activeTab === 'tenants'" 
          :tenants="tenants"
          @update="handleUpdateTenants"
        />
        
        <Settings 
          v-if="activeTab === 'settings'" 
          :modelValue="ownerSettings"
          @update:modelValue="handleUpdateSettings"
        />
      </div>
    </main>

    <!-- Invoice Detail Modal -->
    <InvoicePreviewModal 
      v-if="selectedInvoice" 
      :invoice="selectedInvoice" 
      @close="selectedInvoice = null" 
      @toast="showToast"
    />
  </div>
</template>

<style>
@import "tailwindcss";

@layer base {
  body {
    @apply text-slate-900 bg-slate-50;
  }
}

.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

@media print {
  .no-print { display: none !important; }
}
</style>
