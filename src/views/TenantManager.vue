<script setup>
import { Users, Plus, Edit3, Trash2, X, Building, Banknote, Zap, User as UserIcon } from 'lucide-vue-next';
import { ref } from 'vue';

const props = defineProps(['tenants']);
const emit = defineEmits(['update']);

const showModal = ref(false);
const editingTenant = ref(null);

const openModal = (tenant = null) => {
  if (tenant) {
    editingTenant.value = { ...tenant };
  } else {
    editingTenant.value = {
      id: null,
      roomNumber: '',
      name: '',
      roomRent: 3000,
      electricityPrev: 0,
      waterTenants: 1,
      active: true
    };
  }
  showModal.value = true;
};

const saveTenant = () => {
  let newTenants = [...props.tenants];
  if (editingTenant.value.id) {
    const index = newTenants.findIndex(t => t.id === editingTenant.value.id);
    newTenants[index] = { ...editingTenant.value };
  } else {
    editingTenant.value.id = Date.now().toString();
    newTenants.push({ ...editingTenant.value });
  }
  emit('update', newTenants);
  showModal.value = false;
};

const deleteTenant = (id) => {
  if (confirm('ต้องการลบข้อมูลผู้เช่านี้? (ข้อมูลจะถูกซ่อนจากรายการเบื้องต้น)')) {
    const newTenants = props.tenants.map(t => 
      t.id === id ? { ...t, active: false } : t
    );
    emit('update', newTenants);
  }
};
</script>

<template>
  <div class="max-w-5xl animate-in fade-in slide-in-from-bottom-4 duration-700">
    <div class="flex justify-between items-center mb-10">
        <div>
            <h2 class="text-4xl font-black mb-1 tracking-tight">จัดการห้องพัก</h2>
            <p class="text-slate-400 font-bold uppercase tracking-widest text-[10px]">รายชื่อผู้เช่าปัจจุบัน</p>
        </div>
      <button 
        @click="openModal()"
        class="bg-green-700 text-white px-8 py-4 rounded-2xl font-black flex items-center gap-2 hover:bg-green-800 shadow-2xl shadow-green-200 transition-all hover:scale-105 active:scale-95"
      >
        <Plus class="w-6 h-6" />
        เพิ่มห้องใหม่
      </button>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <div 
        v-for="tenant in tenants.filter(t => t.active)" 
        :key="tenant.id"
        class="bg-white p-8 rounded-[40px] border border-slate-100 hover:shadow-2xl hover:border-green-200 transition-all duration-500 group relative overflow-hidden"
      >
        <div class="absolute -right-10 -top-10 w-40 h-40 bg-slate-50 rounded-full group-hover:bg-green-50 transition-colors duration-500"></div>
        
        <div class="relative z-10">
            <div class="flex items-start justify-between mb-8">
                <div class="flex items-center gap-5">
                    <div class="bg-slate-900 text-white w-20 h-20 rounded-3xl flex flex-col items-center justify-center shadow-xl group-hover:bg-green-700 transition-colors duration-500">
                        <span class="text-[10px] font-black uppercase tracking-widest opacity-40 mb-1 leading-none">ห้อง</span>
                        <span class="text-3xl font-black italic leading-none">{{ tenant.roomNumber }}</span>
                    </div>
                    <div>
                        <h3 class="font-black text-2xl tracking-tight mb-1 group-hover:text-green-800 transition-colors">{{ tenant.name }}</h3>
                        <span class="px-3 py-1 bg-green-50 text-green-700 rounded-full text-[10px] font-black uppercase tracking-wider">กำลังเช่าอยู่</span>
                    </div>
                </div>
            </div>

            <div class="grid grid-cols-3 gap-4 mb-8">
                <div class="bg-slate-50 p-4 rounded-2xl group-hover:bg-white transition-colors duration-500">
                    <Banknote class="w-4 h-4 text-slate-400 mb-2" />
                    <div class="text-[8px] font-black text-slate-400 uppercase mb-0.5">ค่าเช่า</div>
                    <div class="text-sm font-black">{{ tenant.roomRent.toLocaleString() }}</div>
                </div>
                <div class="bg-slate-50 p-4 rounded-2xl group-hover:bg-white transition-colors duration-500">
                    <Zap class="w-4 h-4 text-slate-400 mb-2" />
                    <div class="text-[8px] font-black text-slate-400 uppercase mb-0.5">มิเตอร์</div>
                    <div class="text-sm font-black">{{ tenant.electricityPrev }}</div>
                </div>
                <div class="bg-slate-50 p-4 rounded-2xl group-hover:bg-white transition-colors duration-500">
                    <UserIcon class="w-4 h-4 text-slate-400 mb-2" />
                    <div class="text-[8px] font-black text-slate-400 uppercase mb-0.5">ผู้พัก</div>
                    <div class="text-sm font-black">{{ tenant.waterTenants }} คน</div>
                </div>
            </div>

            <div class="flex gap-3">
                <button 
                    @click="openModal(tenant)"
                    class="flex-1 bg-slate-100 text-slate-600 py-3 rounded-2xl font-bold flex items-center justify-center gap-2 hover:bg-green-50 hover:text-green-700 transition-all border-2 border-transparent hover:border-green-100"
                >
                    <Edit3 class="w-4 h-4" />
                    แก้ไขข้อมูล
                </button>
                <button 
                    @click="deleteTenant(tenant.id)"
                    class="bg-red-50 text-red-400 w-12 h-12 rounded-2xl flex items-center justify-center hover:bg-red-500 hover:text-white transition-all"
                >
                    <Trash2 class="w-5 h-5" />
                </button>
            </div>
        </div>
      </div>
    </div>

    <!-- Tenant Modal -->
    <Transition name="modal">
      <div v-if="showModal" class="fixed inset-0 bg-slate-900/90 backdrop-blur-md z-[100] flex items-center justify-center p-4">
        <div class="bg-white rounded-[40px] w-full max-w-lg p-10 relative shadow-2xl animate-in zoom-in-95 duration-300">
          <button 
            @click="showModal = false"
            class="absolute right-8 top-8 bg-slate-50 w-12 h-12 rounded-2xl flex items-center justify-center text-slate-400 hover:bg-slate-100 transition-all"
          >
            <X class="w-6 h-6" />
          </button>

          <h3 class="text-3xl font-black mb-8 tracking-tight flex items-center gap-3">
              <Building class="w-8 h-8 text-green-700" />
              {{ editingTenant.id ? 'แก้ไขข้อมูล' : 'เพิ่มห้องพัก' }}
          </h3>

          <div class="space-y-8">
            <div class="grid grid-cols-2 gap-6">
              <div class="space-y-2">
                <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest px-2">เลขห้อง</label>
                <input 
                  type="text" 
                  v-model="editingTenant.roomNumber"
                  class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-green-500/10 transition-all" 
                  placeholder="เช่น 101"
                />
              </div>
              <div class="space-y-2">
                <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest px-2">ค่าเช่ารายเดือน</label>
                <input 
                  type="number" 
                  v-model.number="editingTenant.roomRent"
                  class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-green-500/10 transition-all" 
                />
              </div>
            </div>

            <div class="space-y-2">
              <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest px-2">ชื่อผู้เช่า</label>
              <input 
                type="text" 
                v-model="editingTenant.name"
                class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-green-500/10 transition-all" 
                placeholder="ชื่อ-นามสกุล"
              />
            </div>

            <div class="grid grid-cols-2 gap-6">
              <div class="space-y-2">
                <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest px-2">มิเตอร์ไฟ (ปัจจุบัน)</label>
                <input 
                  type="number" 
                  v-model.number="editingTenant.electricityPrev"
                  class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-green-500/10 transition-all text-amber-600" 
                />
              </div>
              <div class="space-y-2">
                <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest px-2">จำนวนผู้พัก (คิดค่าน้ำ)</label>
                <input 
                  type="number" 
                  v-model.number="editingTenant.waterTenants"
                  class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-green-500/10 transition-all text-blue-600" 
                />
              </div>
            </div>

            <button 
              @click="saveTenant"
              class="w-full bg-green-700 text-white py-5 rounded-[24px] font-black text-xl hover:bg-green-800 shadow-2xl shadow-green-100 transition-all hover:scale-[1.02] active:scale-95 flex items-center justify-center gap-3"
            >
              <Plus class="w-6 h-6" v-if="!editingTenant.id" />
              บันทึกข้อมูล
            </button>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.modal-enter-active, .modal-leave-active { transition: all 0.3s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; transform: scale(0.95); }
</style>
