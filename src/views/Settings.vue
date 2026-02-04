<script setup>
import { Settings as SettingsIcon, Building, MapPin, Phone, CreditCard, Zap, Droplets, Globe, CheckCircle2 } from 'lucide-vue-next';

const props = defineProps(['modelValue']);
const emit = defineEmits(['update:modelValue']);

const updateSettings = (key, value) => {
  emit('update:modelValue', { ...props.modelValue, [key]: value });
};
</script>

<template>
  <div class="max-w-4xl animate-in fade-in slide-in-from-bottom-4 duration-700">
    <div class="mb-10">
        <h2 class="text-4xl font-black mb-1 tracking-tight">ตั้งค่าระบบ</h2>
        <p class="text-slate-400 font-bold uppercase tracking-widest text-[10px]">ปรับแต่งข้อมูลหอพักและอัตราค่าบริการ</p>
    </div>
    
    <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        <!-- Main Info -->
        <div class="lg:col-span-2 space-y-8">
            <div class="bg-white p-10 rounded-[40px] shadow-sm border border-slate-100 space-y-8">
                <div class="space-y-4">
                    <div class="flex items-center gap-2 mb-2">
                        <Building class="w-4 h-4 text-green-700" />
                        <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest">ชื่อหอพัก / อพาร์ทเมนต์</label>
                    </div>
                    <input 
                        type="text" 
                        :value="modelValue.apartmentName"
                        @input="updateSettings('apartmentName', $event.target.value)"
                        class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-green-500/10 transition-all text-xl" 
                    />
                </div>

                <div class="space-y-4">
                    <div class="flex items-center gap-2 mb-2">
                        <MapPin class="w-4 h-4 text-green-700" />
                        <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest">ที่อยู่สำหรับการออกบิล</label>
                    </div>
                    <textarea 
                        :value="modelValue.address"
                        @input="updateSettings('address', $event.target.value)"
                        class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-green-500/10 transition-all h-32 resize-none"
                    ></textarea>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                    <div class="space-y-4">
                        <div class="flex items-center gap-2 mb-2">
                            <Phone class="w-4 h-4 text-green-700" />
                            <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest">เบอร์โทรศัพท์ติดต่อ</label>
                        </div>
                        <input 
                            type="text" 
                            :value="modelValue.phoneNumber"
                            @input="updateSettings('phoneNumber', $event.target.value)"
                            class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-green-500/10 transition-all" 
                        />
                    </div>
                    <div class="space-y-4">
                        <div class="flex items-center gap-2 mb-2">
                            <CreditCard class="w-4 h-4 text-green-700" />
                            <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest">ข้อมูลบัญชีรับเงิน</label>
                        </div>
                        <input 
                            type="text" 
                            :value="modelValue.bankInfo"
                            @input="updateSettings('bankInfo', $event.target.value)"
                            class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-bold focus:ring-4 focus:ring-green-500/10 transition-all placeholder:font-normal" 
                            placeholder="ธนาคาร เลขที่บัญชี"
                        />
                    </div>
                </div>
            </div>
        </div>

        <!-- Rates Side -->
        <div class="space-y-8">
            <div class="bg-white p-8 rounded-[40px] shadow-sm border border-slate-100 space-y-8">
                <h3 class="font-black text-sm uppercase tracking-widest text-slate-400 border-b border-slate-50 pb-4 flex items-center gap-2">
                    <SettingsIcon class="w-4 h-4" />
                    อัตราค่าบริการเฉลี่ย
                </h3>

                <div class="space-y-6">
                    <div class="space-y-3">
                        <div class="flex items-center gap-2">
                            <Zap class="w-4 h-4 text-amber-500" />
                            <label class="text-[8px] font-black text-slate-400 uppercase tracking-widest">ค่าไฟ / หน่วย</label>
                        </div>
                        <div class="relative">
                            <input 
                                type="number" 
                                :value="modelValue.defaultElectricityRate"
                                @input="updateSettings('defaultElectricityRate', Number($event.target.value))"
                                class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-black text-2xl focus:ring-4 focus:ring-amber-500/10 transition-all pr-12" 
                            />
                            <span class="absolute right-4 top-1/2 -translate-y-1/2 font-black text-xs text-slate-300">฿</span>
                        </div>
                    </div>

                    <div class="space-y-3">
                        <div class="flex items-center gap-2">
                            <Droplets class="w-4 h-4 text-blue-500" />
                            <label class="text-[8px] font-black text-slate-400 uppercase tracking-widest">ค่าน้ำ / คน</label>
                        </div>
                        <div class="relative">
                            <input 
                                type="number" 
                                :value="modelValue.defaultWaterRate"
                                @input="updateSettings('defaultWaterRate', Number($event.target.value))"
                                class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-black text-2xl focus:ring-4 focus:ring-blue-500/10 transition-all pr-12" 
                            />
                            <span class="absolute right-4 top-1/2 -translate-y-1/2 font-black text-xs text-slate-300">฿</span>
                        </div>
                    </div>

                    <div class="space-y-3">
                        <div class="flex items-center gap-2">
                            <Globe class="w-4 h-4 text-indigo-500" />
                            <label class="text-[8px] font-black text-slate-400 uppercase tracking-widest">ค่าเน็ต (เริ่มต้น)</label>
                        </div>
                        <div class="relative">
                            <input 
                                type="number" 
                                :value="modelValue.defaultInternet"
                                @input="updateSettings('defaultInternet', Number($event.target.value))"
                                class="w-full bg-slate-50 border-0 rounded-2xl p-4 font-black text-2xl focus:ring-4 focus:ring-indigo-500/10 transition-all pr-12" 
                            />
                            <span class="absolute right-4 top-1/2 -translate-y-1/2 font-black text-xs text-slate-300">฿</span>
                        </div>
                    </div>
                </div>

                <div class="pt-4 mt-8 flex flex-col items-center justify-center text-center">
                    <CheckCircle2 class="w-8 h-8 text-green-700 mb-2 opacity-20" />
                    <p class="text-[10px] font-black text-green-700/50 uppercase tracking-tighter">การตั้งค่าทั้งหมดถูกบันทึกอัตโนมัติ</p>
                </div>
            </div>
        </div>
    </div>
  </div>
</template>
