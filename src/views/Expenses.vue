<script setup>
import { ref, computed } from 'vue';
import { Wallet, Plus, Trash2, Calendar, Tag, Banknote, Search, X } from 'lucide-vue-next';

const props = defineProps(['expenses', 'isProcessing']);
const emit = defineEmits(['save', 'delete']);

const showModal = ref(false);
const searchQuery = ref('');
const newExpense = ref({
    title: '',
    amount: 0,
    category: 'ซ่อมบำรุง',
    date: new Date().toISOString().split('T')[0]
});

const categories = ['ซ่อมบำรุง', 'ค่าน้ำ-ไฟส่วนกลาง', 'แม่บ้าน', 'ภาษี', 'อื่นๆ'];

const filteredExpenses = computed(() => {
    if (!searchQuery.value) return props.expenses;
    const q = searchQuery.value.toLowerCase();
    return props.expenses.filter(ex => 
        ex.title.toLowerCase().includes(q) || 
        ex.category.toLowerCase().includes(q)
    );
});

const totalExpenses = computed(() => {
    return filteredExpenses.value.reduce((sum, ex) => sum + (ex.amount || 0), 0);
});

const saveExpense = () => {
    if (!newExpense.value.title || newExpense.value.amount <= 0) return;
    emit('save', { ...newExpense.value });
    showModal.value = false;
    newExpense.value = {
        title: '',
        amount: 0,
        category: 'ซ่อมบำรุง',
        date: new Date().toISOString().split('T')[0]
    };
};

const formatDate = (dateStr) => {
    const d = new Date(dateStr);
    return d.toLocaleDateString('th-TH', { day: 'numeric', month: 'short', year: '2-digit' });
};
</script>

<template>
  <div class="max-w-5xl animate-in fade-in slide-in-from-bottom-4 duration-700">
    <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-6 mb-10">
        <div>
            <h2 class="text-4xl font-black mb-1 tracking-tight">บันทึกรายจ่าย</h2>
            <p class="text-slate-400 font-bold uppercase tracking-widest text-[10px]">จัดการต้นทุนและค่าใช้จ่ายหอพัก</p>
        </div>
        <button 
            @click="showModal = true"
            class="bg-red-600 text-white px-8 py-4 rounded-2xl font-black flex items-center gap-2 hover:bg-red-700 shadow-2xl shadow-red-100 transition-all hover:scale-105 active:scale-95"
        >
            <Plus class="w-6 h-6" />
            เพิ่มรายจ่าย
        </button>
    </div>

    <!-- Summary Card -->
    <div class="bg-white p-8 rounded-[40px] border border-slate-100 mb-10 flex items-center justify-between shadow-sm">
        <div class="flex items-center gap-6">
            <div class="bg-red-50 p-4 rounded-3xl text-red-600">
                <Wallet class="w-10 h-10" />
            </div>
            <div>
                <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-1">รวมรายจ่ายทั้งหมด</p>
                <h3 class="text-4xl font-black text-slate-900 tracking-tight">฿{{ totalExpenses.toLocaleString() }}</h3>
            </div>
        </div>
        <div class="relative w-64 hidden md:block">
            <Search class="absolute left-4 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-300" />
            <input 
                v-model="searchQuery"
                type="text" 
                placeholder="ค้นหาชื่อรายการ..."
                class="w-full bg-slate-50 border-0 rounded-2xl py-3 pl-10 pr-4 font-bold text-sm focus:ring-4 focus:ring-red-500/10 transition-all"
            />
        </div>
    </div>

    <!-- Expense List -->
    <div class="space-y-4">
        <div 
            v-for="expense in filteredExpenses" 
            :key="expense.id"
            class="bg-white p-6 rounded-[28px] border border-slate-100 flex items-center justify-between group hover:border-red-200 transition-all duration-300"
        >
            <div class="flex items-center gap-6">
                <div class="bg-slate-50 w-14 h-14 rounded-2xl flex items-center justify-center text-slate-400 group-hover:bg-red-50 group-hover:text-red-500 transition-colors">
                    <Tag class="w-6 h-6" />
                </div>
                <div>
                    <h4 class="font-black text-xl text-slate-900 mb-1">{{ expense.title }}</h4>
                    <div class="flex items-center gap-4 text-[10px] font-black uppercase tracking-widest text-slate-400">
                        <span class="bg-slate-100 px-2 py-1 rounded-lg">{{ expense.category }}</span>
                        <span class="flex items-center gap-1"><Calendar class="w-3 h-3" /> {{ formatDate(expense.date) }}</span>
                    </div>
                </div>
            </div>
            
            <div class="flex items-center gap-6">
                <span class="text-2xl font-black text-red-600 italic">- ฿{{ expense.amount.toLocaleString() }}</span>
                <button 
                    @click="emit('delete', expense.id)"
                    class="w-12 h-12 bg-slate-50 text-slate-300 rounded-2xl flex items-center justify-center hover:bg-red-500 hover:text-white transition-all shadow-sm"
                >
                    <Trash2 class="w-5 h-5" />
                </button>
            </div>
        </div>

        <div v-if="filteredExpenses.length === 0" class="text-center py-20 bg-slate-50 rounded-[40px] border-2 border-dashed border-slate-200">
            <p class="text-slate-400 font-black uppercase tracking-widest">ไม่พบรายการใช้จ่าย</p>
        </div>
    </div>

    <!-- Modal -->
    <Transition name="modal">
        <div v-if="showModal" class="fixed inset-0 bg-slate-900/90 backdrop-blur-md z-[100] flex items-center justify-center p-4">
            <div class="bg-white rounded-[40px] w-full max-w-lg p-10 relative shadow-2xl animate-in zoom-in-95 duration-300">
                <button 
                    @click="showModal = false"
                    class="absolute right-8 top-8 bg-slate-50 w-12 h-12 rounded-2xl flex items-center justify-center text-slate-400 hover:bg-slate-100 transition-all"
                >
                    <X class="w-6 h-6" />
                </button>

                <h3 class="text-3xl font-black mb-8 tracking-tight flex items-center gap-3 text-red-600">
                    <Banknote class="w-8 h-8" />
                    เพิ่มรายการรายจ่าย
                </h3>

                <div class="space-y-6">
                    <div class="space-y-2">
                        <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest px-2">ชื่อรายการ</label>
                        <input 
                            v-model="newExpense.title"
                            type="text"
                            placeholder="เช่น ค่าซ่อมก๊อกน้ำ ห้อง 101"
                            class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-red-500/10 transition-all"
                        />
                    </div>

                    <div class="grid grid-cols-2 gap-6">
                        <div class="space-y-2">
                            <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest px-2">จำนวนเงิน (บาท)</label>
                            <input 
                                v-model.number="newExpense.amount"
                                type="number"
                                class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-red-500/10 transition-all text-red-600"
                            />
                        </div>
                        <div class="space-y-2">
                            <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest px-2">วันที่</label>
                            <input 
                                v-model="newExpense.date"
                                type="date"
                                class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-red-500/10 transition-all"
                            />
                        </div>
                    </div>

                    <div class="space-y-2">
                        <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest px-2">หมวดหมู่</label>
                        <select 
                            v-model="newExpense.category"
                            class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-red-500/10 transition-all outline-none"
                        >
                            <option v-for="cat in categories" :key="cat" :value="cat">{{ cat }}</option>
                        </select>
                    </div>

                    <button 
                        @click="saveExpense"
                        :disabled="isProcessing"
                        class="w-full bg-red-600 text-white py-5 rounded-[24px] font-black text-xl hover:bg-red-700 shadow-2xl shadow-red-100 transition-all hover:scale-[1.02] active:scale-95 flex items-center justify-center gap-3 disabled:opacity-50"
                    >
                        <Plus class="w-6 h-6" v-if="!isProcessing" />
                        <span v-else class="animate-spin rounded-full h-5 w-5 border-b-2 border-white"></span>
                        {{ isProcessing ? 'กำลังบันทึก...' : 'บันทึกรายจ่าย' }}
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
