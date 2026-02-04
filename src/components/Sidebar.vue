<script setup>
import { LayoutDashboard, PlusCircle, History, Users, Settings, Building2 } from 'lucide-vue-next';

const props = defineProps(['modelValue', 'apartmentName']);
const emit = defineEmits(['update:modelValue']);

const navItems = [
  { id: 'dashboard', label: 'แดชบอร์ด', icon: LayoutDashboard },
  { id: 'create', label: 'ออกบิล', icon: PlusCircle },
  { id: 'history', label: 'ประวัติ', icon: History },
  { id: 'tenants', label: 'จัดการห้อง', icon: Users },
  { id: 'settings', label: 'ตั้งค่า', icon: Settings },
];
</script>

<template>
  <nav class="fixed bottom-0 w-full md:left-0 md:top-0 md:w-64 md:h-full bg-white border-t md:border-t-0 md:border-r border-slate-200 z-50">
    <div class="p-6 hidden md:block border-b border-slate-100">
      <div class="flex items-center gap-3">
        <div class="bg-green-700 p-2 rounded-xl text-white shadow-lg shadow-green-200">
          <Building2 class="w-6 h-6" />
        </div>
        <div>
          <span class="font-bold text-xl block leading-none">SmartRent</span>
          <span class="text-[10px] text-slate-400 font-bold uppercase tracking-wider">Dormitory System</span>
        </div>
      </div>
    </div>
    
    <div class="flex md:flex-col justify-around md:justify-start p-4 md:p-6 gap-2 md:gap-4">
      <button 
        v-for="item in navItems"
        :key="item.id"
        @click="emit('update:modelValue', item.id)"
        :class="[
          'flex items-center gap-3 p-3 rounded-2xl transition-all duration-300 group',
          modelValue === item.id 
            ? 'bg-green-700 text-white shadow-lg shadow-green-100' 
            : 'text-slate-400 hover:bg-slate-50 hover:text-slate-600'
        ]"
      >
        <component 
          :is="item.icon" 
          :class="['w-5 h-5', modelValue === item.id ? 'text-white' : 'group-hover:scale-110 transition-transform']" 
        />
        <span class="hidden md:inline font-bold">{{ item.label }}</span>
      </button>
    </div>
  </nav>
</template>
