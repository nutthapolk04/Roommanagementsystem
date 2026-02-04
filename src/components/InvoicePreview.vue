<script setup>
import { Building2, User, Calendar, Zap, Droplets, Globe, CreditCard, CheckCircle2 } from 'lucide-vue-next';
const props = defineProps(['data']);
</script>

<template>
  <div class="bg-white p-12 max-w-[850px] mx-auto text-slate-800 font-sans leading-relaxed shadow-2xl rounded-[40px] border border-slate-100" id="invoice-content">
    <!-- Header with Branding -->
    <div class="flex justify-between items-start mb-12 border-b border-slate-100 pb-10">
      <div>
        <div class="flex items-center gap-3 mb-4">
            <div class="bg-green-700 p-3 rounded-2xl text-white shadow-lg shadow-green-100">
                <Building2 class="w-8 h-8" />
            </div>
            <div>
                <h1 class="text-3xl font-black tracking-tight text-slate-900 leading-none">SmartRent</h1>
                <p class="text-[10px] font-black uppercase tracking-[0.2em] text-slate-400 mt-1">Premium Living Management</p>
            </div>
        </div>
        <div class="text-sm font-bold text-slate-500 max-w-xs">{{ data.apartmentName }}</div>
        <div class="text-xs text-slate-400 mt-1 max-w-xs leading-relaxed">{{ data.address }}</div>
        <div class="flex items-center gap-2 mt-3 text-xs font-black text-slate-800">
            <div class="bg-slate-100 p-1 rounded-md"><CreditCard class="w-3 h-3" /></div>
            {{ data.phoneNumber }}
        </div>
      </div>
      
      <div class="text-right">
        <h2 class="text-5xl font-black text-slate-900 tracking-tighter mb-2">INVOICE</h2>
        <div class="bg-slate-900 text-white px-4 py-2 rounded-xl inline-block font-black text-xs uppercase tracking-widest mb-4">
            {{ data.month }} {{ data.year }}
        </div>
        <div class="text-[10px] font-black text-slate-400 uppercase tracking-widest">Date Issued</div>
        <div class="text-sm font-bold">{{ new Date(data.createdAt || Date.now()).toLocaleDateString('th-TH') }}</div>
      </div>
    </div>

    <!-- Info Section -->
    <div class="grid grid-cols-2 gap-12 mb-12">
      <div class="space-y-4">
        <div class="flex items-center gap-2 text-[10px] font-black text-slate-400 uppercase tracking-widest border-b border-slate-50 pb-2">
            <User class="w-3 h-3" />
            Billed To
        </div>
        <div>
            <div class="text-xl font-black text-slate-900">{{ data.tenantName }}</div>
            <div class="flex items-center gap-2 mt-1">
                <span class="text-xs font-bold text-slate-500 uppercase tracking-widest">Room Number:</span>
                <span class="bg-green-50 text-green-700 px-3 py-1 rounded-full text-xs font-black">{{ data.roomNumber }}</span>
            </div>
        </div>
      </div>
      
      <div class="space-y-4">
        <div class="flex items-center gap-2 text-[10px] font-black text-slate-400 uppercase tracking-widest border-b border-slate-50 pb-2">
            <CreditCard class="w-3 h-3" />
            Payment Details
        </div>
        <div class="bg-slate-50 p-4 rounded-2xl border border-slate-100">
            <div class="text-[10px] font-black text-slate-400 uppercase mb-1">Bank Account Info</div>
            <div class="text-sm font-black text-slate-900">{{ data.bankInfo }}</div>
        </div>
      </div>
    </div>

    <!-- Line Items -->
    <div class="mb-12">
        <div class="grid grid-cols-12 gap-4 px-6 mb-4 text-[10px] font-black text-slate-400 uppercase tracking-widest">
            <div class="col-span-7 italic">Description</div>
            <div class="col-span-2 text-center italic">Details</div>
            <div class="col-span-3 text-right italic">Amount (THB)</div>
        </div>
        
        <div class="space-y-2">
            <!-- Room Rent -->
            <div class="grid grid-cols-12 gap-4 bg-white p-6 rounded-3xl border border-slate-100 items-center">
                <div class="col-span-7">
                    <div class="font-black text-slate-900 text-lg">ค่าเช่าห้อง (Room Rent)</div>
                    <div class="text-xs text-slate-400 font-bold uppercase tracking-wider">Base monthly rental fee</div>
                </div>
                <div class="col-span-2 text-center text-xs font-bold text-slate-400 uppercase tracking-widest">รายเดือน</div>
                <div class="col-span-3 text-right font-black text-xl italic">{{ data.roomRent?.toLocaleString() }}</div>
            </div>

            <!-- Electricity -->
            <div class="grid grid-cols-12 gap-4 bg-white p-6 rounded-3xl border border-slate-100 items-center relative overflow-hidden group">
                <div class="absolute left-0 top-0 bottom-0 w-1 bg-amber-400 opacity-20"></div>
                <div class="col-span-7">
                    <div class="flex items-center gap-2">
                        <Zap class="w-4 h-4 text-amber-500" />
                        <div class="font-black text-slate-900 text-lg">ค่าไฟฟ้า (Electricity)</div>
                    </div>
                    <div class="flex gap-4 mt-2">
                        <div class="text-[10px] text-slate-400 font-bold uppercase tracking-widest">
                            Prev: <span class="text-slate-900">{{ data.electricityPrev }}</span>
                        </div>
                        <div class="text-[10px] text-slate-400 font-bold uppercase tracking-widest">
                            Curr: <span class="text-slate-900">{{ data.electricityCurrent }}</span>
                        </div>
                    </div>
                </div>
                <div class="col-span-2 text-center">
                    <div class="text-lg font-black text-amber-600">{{ data.electricityUnits }}</div>
                    <div class="text-[8px] font-black text-slate-400 uppercase tracking-tighter">Units Used</div>
                </div>
                <div class="col-span-3 text-right font-black text-xl italic">
                    <div class="text-xs font-bold text-slate-400 mb-1">@{{ data.electricityRate }} ฿</div>
                    {{ data.electricityTotal?.toLocaleString() }}
                </div>
            </div>

            <!-- Water -->
            <div class="grid grid-cols-12 gap-4 bg-white p-6 rounded-3xl border border-slate-100 items-center relative overflow-hidden">
                <div class="absolute left-0 top-0 bottom-0 w-1 bg-blue-400 opacity-20"></div>
                <div class="col-span-7">
                    <div class="flex items-center gap-2">
                        <Droplets class="w-4 h-4 text-blue-500" />
                        <div class="font-black text-slate-900 text-lg">ค่าน้ำ (Water Charge)</div>
                    </div>
                    <div class="text-xs text-slate-400 font-bold uppercase tracking-wider">Fixed fee based on occupants</div>
                </div>
                <div class="col-span-2 text-center">
                    <div class="text-lg font-black text-blue-600">{{ data.waterTenants }}</div>
                    <div class="text-[8px] font-black text-slate-400 uppercase tracking-tighter">Occupants</div>
                </div>
                <div class="col-span-3 text-right font-black text-xl italic">
                    <div class="text-xs font-bold text-slate-400 mb-1">@{{ data.waterRate }} ฿</div>
                    {{ data.waterTotal?.toLocaleString() }}
                </div>
            </div>

            <!-- Internet -->
            <div v-if="data.internet > 0" class="grid grid-cols-12 gap-4 bg-white p-6 rounded-3xl border border-slate-100 items-center">
                <div class="col-span-7">
                    <div class="flex items-center gap-2">
                        <Globe class="w-4 h-4 text-indigo-500" />
                        <div class="font-black text-slate-900 text-lg">ค่าอินเทอร์เน็ต (Internet)</div>
                    </div>
                    <div class="text-xs text-slate-400 font-bold uppercase tracking-wider">Monthly high-speed connection</div>
                </div>
                <div class="col-span-2 text-center text-xs font-bold text-slate-400 uppercase tracking-widest">Unlimited</div>
                <div class="col-span-3 text-right font-black text-xl italic">{{ data.internet?.toLocaleString() }}</div>
            </div>
        </div>
    </div>

    <!-- Grand Total & Payment Terms -->
    <div class="flex flex-col md:flex-row justify-between items-end gap-12">
        <div class="max-w-xs">
            <div class="bg-green-50 p-6 rounded-[32px] border border-green-100 relative overflow-hidden">
                <CheckCircle2 class="absolute -right-4 -bottom-4 w-24 h-24 text-green-200 opacity-20" />
                <p class="text-[10px] font-black text-green-800 uppercase tracking-[0.2em] mb-3">Notice / หมายเหตุ</p>
                <p class="text-[11px] font-bold text-green-700 leading-relaxed italic">
                    กรุณาชำระเงินภายในวันที่ 5 ของเดือน หากล่าช้าอาจมีค่าปรับตามระเบียบที่กำหนด ขอบคุณที่ร่วมเป็นส่วนหนึ่งของครอบครัว SmartRent
                </p>
            </div>
            <div class="mt-6 flex gap-4 px-4 overflow-hidden">
                <div class="text-[8px] font-black text-slate-300 uppercase tracking-widest rotate-90 origin-left translate-y-8">SmartRent Official</div>
                <div class="border-l-2 border-slate-100 pl-4 py-1 text-[9px] text-slate-400 italic font-serif">
                    This is an electronically generated invoice and does not require a physical signature.
                </div>
            </div>
        </div>

        <div class="text-right flex-1 bg-slate-900 p-10 rounded-[40px] shadow-2xl shadow-slate-200 relative overflow-hidden">
            <div class="absolute -right-10 -top-10 w-40 h-40 bg-white opacity-5 rounded-full"></div>
            <div class="relative z-10">
                <div class="text-[10px] font-black text-white/40 uppercase tracking-[0.3em] mb-2 px-1">Total Amount Due</div>
                <div class="text-6xl font-black text-white tracking-tighter leading-none mb-1">
                    {{ data.grandTotal?.toLocaleString() }}
                </div>
                <div class="text-sm font-black text-green-400 uppercase tracking-widest italic">Thai Baht (฿)</div>
            </div>
        </div>
    </div>
  </div>
</template>

<style scoped>
#invoice-content {
    font-family: 'Prompt', 'Inter', sans-serif;
}

@media print {
  #invoice-content {
    width: 100% !important;
    max-width: none !important;
    padding: 2cm !important;
    border: none !important;
    box-shadow: none !important;
    border-radius: 0 !important;
  }
}
</style>
