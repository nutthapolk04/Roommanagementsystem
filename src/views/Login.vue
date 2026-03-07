<script setup>
import { ref } from 'vue';
import { auth } from '../firebase';
import { signInWithEmailAndPassword, sendPasswordResetEmail } from 'firebase/auth';
import { LogIn, Mail, Lock, Building, Loader2 } from 'lucide-vue-next';

const email = ref('');
const password = ref('');
const loading = ref(false);
const error = ref('');

const handleLogin = async () => {
  if (!email.value || !password.value) {
    error.value = 'กรุณากรอกอีเมลและรหัสผ่าน';
    return;
  }
  
  loading.value = true;
  error.value = '';
  
  try {
    await signInWithEmailAndPassword(auth, email.value, password.value);
  } catch (err) {
    console.error('Login error:', err);
    if (err.code === 'auth/user-not-found' || err.code === 'auth/wrong-password' || err.code === 'auth/invalid-credential') {
      error.value = 'อีเมลหรือรหัสผ่านไม่ถูกต้อง';
    } else {
      error.value = 'เกิดข้อผิดพลาดในการเข้าสู่ระบบ กรุณาลองใหม่';
    }
  } finally {
    loading.value = false;
  }
};

const handleForgotPassword = async () => {
    if (!email.value) {
        error.value = 'กรุณากรอกอีเมลเพื่อรีเซ็ตรหัสผ่าน';
        return;
    }
    try {
        await sendPasswordResetEmail(auth, email.value);
        alert('ส่งลิงก์รีเซ็ตรหัสผ่านไปที่อีเมลของคุณแล้ว');
    } catch (err) {
        error.value = 'ไม่สามารถส่งลิงก์รีเซ็ตรหัสผ่านได้';
    }
}
</script>

<template>
  <div class="min-h-screen bg-slate-50 flex items-center justify-center p-6 bg-[radial-gradient(circle_at_top_right,_var(--tw-gradient-stops))] from-green-50 via-slate-50 to-indigo-50">
    <div class="w-full max-w-md animate-in fade-in zoom-in duration-700">
      
      <!-- Logo & Header -->
      <div class="text-center mb-10">
        <div class="bg-green-700 w-16 h-16 rounded-[24px] flex items-center justify-center mx-auto mb-6 shadow-2xl shadow-green-200 rotate-3 transform transition-transform hover:rotate-0">
          <Building class="text-white w-8 h-8" />
        </div>
        <h1 class="text-4xl font-black tracking-tight text-slate-900 mb-2 italic">พี่ปุ้ยห้องพัก</h1>
        <p class="text-slate-400 font-bold uppercase tracking-[0.2em] text-[10px]">Premium Room Management System</p>
      </div>

      <!-- Login Card -->
      <div class="bg-white p-10 rounded-[48px] shadow-2xl border border-white relative overflow-hidden">
        <!-- Decoration -->
        <div class="absolute -right-10 -top-10 w-40 h-40 bg-green-50 rounded-full blur-3xl opacity-50"></div>
        
        <form @submit.prevent="handleLogin" class="relative z-10 space-y-6">
          <div class="space-y-2">
            <label class="text-[10px] font-black uppercase tracking-widest text-slate-400 px-2 flex items-center gap-2">
              <Mail class="w-3 h-3" /> อีเมลผู้ดูแล
            </label>
            <input 
              v-model="email"
              type="email" 
              required
              placeholder="admin@puiroom.com"
              class="w-full bg-slate-50 border-2 border-transparent focus:border-green-500 focus:bg-white rounded-2xl p-4 font-bold transition-all outline-none"
            />
          </div>

          <div class="space-y-2">
            <div class="flex justify-between items-center px-2">
                <label class="text-[10px] font-black uppercase tracking-widest text-slate-400 flex items-center gap-2">
                  <Lock class="w-3 h-3" /> รหัสผ่าน
                </label>
                <button type="button" @click="handleForgotPassword" class="text-[10px] font-black uppercase tracking-widest text-green-700 hover:text-green-800">ลืมรหัสผ่าน?</button>
            </div>
            <input 
              v-model="password"
              type="password" 
              required
              placeholder="••••••••"
              class="w-full bg-slate-50 border-2 border-transparent focus:border-green-500 focus:bg-white rounded-2xl p-4 font-bold transition-all outline-none"
            />
          </div>

          <div v-if="error" class="bg-red-50 text-red-500 p-4 rounded-xl text-xs font-bold border border-red-100 animate-shake">
            {{ error }}
          </div>

          <button 
            type="submit"
            :disabled="loading"
            class="w-full bg-green-700 text-white font-black py-4 rounded-2xl shadow-xl shadow-green-100 hover:bg-green-800 hover:scale-[1.02] active:scale-95 transition-all flex items-center justify-center gap-3 disabled:opacity-50 disabled:scale-100"
          >
            <template v-if="loading">
              <Loader2 class="w-5 h-5 animate-spin" />
              <span>กำลังเข้าสู่ระบบ...</span>
            </template>
            <template v-else>
              <LogIn class="w-5 h-5" />
              <span>เข้าสู่ระบบ</span>
            </template>
          </button>
        </form>
      </div>

      <!-- Footer Info -->
      <p class="text-center mt-10 text-slate-300 font-bold text-xs uppercase tracking-widest">
        &copy; 2026 Pui Room Management • V 3.1.0
      </p>
    </div>
  </div>
</template>

<style scoped>
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-4px); }
  75% { transform: translateX(4px); }
}
.animate-shake {
  animation: shake 0.2s ease-in-out 0s 2;
}
</style>
