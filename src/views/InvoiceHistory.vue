<script setup>
import { ReceiptText, Trash2, Eye, Search, ChevronDown, ChevronRight, CalendarDays, Wallet, Check, X, Loader2 } from 'lucide-vue-next';
import { ref, computed } from 'vue';

const props = defineProps(['invoices']);
const emit = defineEmits(['view', 'delete', 'toggle-status']);

const loadingIds = ref(new Set());

const handleToggleStatus = async (id, currentStatus) => {
    loadingIds.value.add(id);
    try {
        await emit('toggle-status', id, !currentStatus);
    } finally {
        setTimeout(() => {
            loadingIds.value.delete(id);
        }, 500);
    }
};

const searchQuery = ref('');
const expandedGroups = ref({});

const filteredInvoices = computed(() => {
  if (!searchQuery.value) return props.invoices;
  const q = searchQuery.value.toLowerCase();
  return props.invoices.filter(inv => 
    inv.roomNumber?.toLowerCase().includes(q) || 
    inv.tenantName?.toLowerCase().includes(q) ||
    inv.month?.toLowerCase().includes(q)
  );
});

const groupedInvoices = computed(() => {
    const groups = {};
    filteredInvoices.value.forEach(inv => {
        const key = `${inv.month} ${inv.year}`;
        if (!groups[key]) {
            groups[key] = {
                id: key,
                label: key,
                items: [],
                total: 0
            };
            // Default expanded if not set
            if (expandedGroups.value[key] === undefined) {
                expandedGroups.value[key] = true;
            }
        }
        groups[key].items.push(inv);
        groups[key].total += (inv.grandTotal || 0);
    });
    
    // Convert to array and sort (most recent first)
    return Object.values(groups).sort((a, b) => {
        // Sort by first item's createdAt as a proxy for the group's time
        const timeA = a.items[0]?.createdAt || 0;
        const timeB = b.items[0]?.createdAt || 0;
        return timeB - timeA;
    });
});

const toggleGroup = (key) => {
    expandedGroups.value[key] = !expandedGroups.value[key];
};

const formatDate = (ts) => {
    if (!ts) return 'N/A';
    return new Intl.DateTimeFormat('th-TH', { 
        day: 'numeric', 
        month: 'short', 
        year: '2-digit',
        hour: '2-digit',
        minute: '2-digit'
    }).format(new Date(ts));
};
</script>

<template>
  <div class="max-w-5xl animate-in fade-in slide-in-from-bottom-4 duration-700">
    <!-- Header Section -->
    <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-6 mb-12">
        <div>
            <h2 class="text-4xl font-black mb-1 tracking-tight">ประวัติการออกบิล</h2>
            <div class="flex items-center gap-2 text-slate-400 font-bold uppercase tracking-widest text-[10px]">
                <CalendarDays class="w-3 h-3" />
                จัดกลุ่มตามรายเดือน
            </div>
        </div>
        
        <div class="relative w-full md:w-96 group">
            <Search class="absolute left-5 top-1/2 -translate-y-1/2 w-5 h-5 text-slate-300 group-focus-within:text-green-600 transition-colors" />
            <input 
                v-model="searchQuery"
                type="text" 
                placeholder="ค้นหาเลขห้อง, ชื่อผู้เช่า, หรือเดือน..." 
                class="w-full bg-white border-2 border-slate-100 rounded-[28px] py-5 pl-14 pr-6 font-bold focus:border-green-500 focus:ring-0 transition-all shadow-sm placeholder:font-medium"
            />
        </div>
    </div>
    
    <div v-if="groupedInvoices.length === 0" class="bg-white p-24 text-center rounded-[48px] border-4 border-dashed border-slate-100">
      <div class="bg-slate-50 w-24 h-24 rounded-full flex items-center justify-center mx-auto mb-6">
          <ReceiptText class="w-10 h-10 text-slate-200" />
      </div>
      <p class="text-slate-300 font-black uppercase tracking-widest text-lg">ไม่พบข้อมูลการออกบิล</p>
    </div>

    <div v-else class="space-y-10">
      <div 
        v-for="group in groupedInvoices" 
        :key="group.id"
        class="animate-in fade-in slide-in-from-bottom-2 duration-500"
      >
        <!-- Group Header -->
        <div 
            @click="toggleGroup(group.id)"
            class="flex items-center justify-between p-6 bg-white border border-slate-100 rounded-[32px] cursor-pointer hover:border-green-200 transition-all shadow-sm mb-4"
        >
            <div class="flex items-center gap-4">
                <div :class="['w-10 h-10 rounded-2xl flex items-center justify-center transition-colors', expandedGroups[group.id] ? 'bg-green-700 text-white' : 'bg-slate-100 text-slate-400']">
                    <component :is="expandedGroups[group.id] ? ChevronDown : ChevronRight" class="w-6 h-6" />
                </div>
                <div>
                    <h3 class="font-black text-xl tracking-tight">{{ group.label }}</h3>
                    <div class="flex items-center gap-3 text-[10px] font-black uppercase tracking-widest text-slate-400">
                        <span>รายการ: {{ group.items.length }}</span>
                        <span class="w-1 h-1 bg-slate-200 rounded-full"></span>
                        <span class="text-green-700">รวม: ฿{{ group.total.toLocaleString() }}</span>
                    </div>
                </div>
            </div>
            <div class="hidden sm:flex items-center gap-4 pr-4">
                <Wallet class="w-5 h-5 text-slate-200" />
            </div>
        </div>

        <!-- Group Items -->
        <Transition name="expand">
            <div v-show="expandedGroups[group.id]" class="space-y-4 pl-4 border-l-2 border-slate-50 ml-10 overflow-hidden">
                <div 
                    v-for="inv in group.items" 
                    :key="inv.id"
                    class="bg-white p-6 rounded-[28px] border border-slate-100 flex flex-col md:flex-row items-center justify-between hover:shadow-xl hover:border-green-300 transition-all duration-300 group"
                >
                    <div class="flex items-center gap-6 mb-6 md:mb-0 w-full md:w-auto">
                        <div class="bg-slate-50 w-16 h-16 rounded-[20px] flex flex-col items-center justify-center group-hover:bg-green-700 group-hover:text-white transition-all duration-300">
                            <span class="text-[8px] font-black uppercase tracking-widest opacity-40 leading-none mb-1">ห้อง</span>
                            <span class="text-xl font-black italic leading-none">{{ inv.roomNumber }}</span>
                        </div>
                        <div class="flex-1">
                            <div class="flex flex-wrap items-center gap-3 mb-2">
                                <h3 class="font-black text-2xl tracking-tight leading-none text-slate-900">{{ inv.tenantName }}</h3>
                                <span :class="[
                                    'text-[10px] font-black uppercase tracking-[0.2em] px-3 py-1.5 rounded-xl whitespace-nowrap shadow-sm border',
                                    inv.type === 'move-in' ? 'bg-indigo-600 text-white border-indigo-700' : 'bg-green-700 text-white border-green-800'
                                ]">
                                    {{ inv.type === 'move-in' ? 'บิลแรกเข้า' : 'บิลรายเดือน' }}
                                </span>
                                
                                <!-- Payment Status Badge -->
                                <button 
                                    @click.stop="handleToggleStatus(inv.id, inv.paid)"
                                    :disabled="loadingIds.has(inv.id)"
                                    :class="[
                                        'flex items-center gap-2 px-3 py-1.5 rounded-xl text-[10px] font-black uppercase tracking-widest transition-all shadow-sm border-2',
                                        inv.paid 
                                            ? 'bg-emerald-50 text-emerald-700 border-emerald-100 hover:bg-emerald-100' 
                                            : 'bg-amber-50 text-amber-600 border-amber-100 hover:bg-amber-100'
                                    ]"
                                >
                                    <template v-if="loadingIds.has(inv.id)">
                                        <Loader2 class="w-3 h-3 animate-spin" />
                                    </template>
                                    <template v-else>
                                        <Check v-if="inv.paid" class="w-3 h-3" />
                                        <X v-else class="w-3 h-3" />
                                    </template>
                                    {{ inv.paid ? 'ชำระแล้ว' : 'รอชำระ' }}
                                </button>
                            </div>
                            <div class="flex items-center gap-4 text-xs font-bold uppercase tracking-wider">
                                <span class="text-blue-600 font-black text-base italic">฿{{ inv.grandTotal?.toLocaleString() }}</span>
                                <span class="w-1.5 h-1.5 bg-slate-200 rounded-full"></span>
                                <span class="text-slate-400 group-hover:text-slate-600">{{ formatDate(inv.createdAt) }}</span>
                            </div>
                        </div>
                    </div>

                    <div class="flex items-center gap-3 w-full md:w-auto">
                        <button 
                            @click="emit('view', inv)"
                            class="flex-1 md:flex-none h-12 px-6 rounded-2xl bg-green-50 text-green-700 font-black text-sm flex items-center justify-center gap-2 hover:bg-green-700 hover:text-white transition-all shadow-sm"
                        >
                            <Eye class="w-4 h-4" />
                            ดูบิล
                        </button>
                        <button 
                            @click="emit('delete', inv.id)"
                            class="w-12 h-12 rounded-2xl bg-red-50 text-red-300 flex items-center justify-center hover:bg-red-500 hover:text-white transition-all"
                        >
                            <Trash2 class="w-5 h-5" />
                        </button>
                    </div>
                </div>
            </div>
        </Transition>
      </div>
    </div>
  </div>
</template>

<style scoped>
.expand-enter-active, .expand-leave-active {
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    max-height: 2000px;
}
.expand-enter-from, .expand-leave-to {
    max-height: 0;
    opacity: 0;
    transform: translateY(-10px);
}
</style>
