<template>
  <main class="flex items-center justify-center w-full min-h-screen relative overflow-hidden bg-[#00000F]">
    <!-- Space Background -->
    <div class="absolute inset-0 z-0">
      <div class="stars-layer"></div>
      <div class="scanlines"></div>
    </div>

    <!-- Glass Portal -->
    <section class="relative z-10 w-full max-w-md p-10 m-4 glass-card animate-entry">
      <div class="flex flex-col items-center justify-center mb-10">
        <!-- Logo or Icon -->
        <div class="w-16 h-16 rounded-full bg-cyan-500/10 flex items-center justify-center border border-cyan-400/30 mb-4 shadow-[0_0_30px_rgba(0,255,255,0.2)]">
           <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 text-cyan-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
             <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
           </svg>
        </div>
        <h2 class="text-4xl font-bold text-center text-white tracking-widest uppercase glow-text font-display">
          {{ globalConfig.installationName }}
        </h2>
        <div class="h-1 w-24 bg-cyan-500/50 mt-4 rounded-full"></div>
      </div>

      <form class="space-y-6" @submit.prevent="submitFormLogin">
        <div class="space-y-5">
          <div class="group relative">
            <input v-model="credentials.email" type="text" placeholder=" "
              class="peer w-full px-4 py-4 bg-black/40 border border-cyan-900/50 rounded-lg text-white placeholder-transparent focus:outline-none focus:border-cyan-400 focus:ring-1 focus:ring-cyan-400 transition-all duration-300" required />
            <label class="absolute left-4 -top-2.5 bg-[#050A28] px-1 text-xs text-cyan-400 transition-all peer-placeholder-shown:text-base peer-placeholder-shown:text-gray-400 peer-placeholder-shown:top-4 peer-focus:-top-2.5 peer-focus:text-cyan-400 peer-focus:text-xs">
              IDENTIDADE (Email)
            </label>
          </div>

          <div class="group relative">
             <input v-model="credentials.password" type="password" placeholder=" "
              class="peer w-full px-4 py-4 bg-black/40 border border-cyan-900/50 rounded-lg text-white placeholder-transparent focus:outline-none focus:border-cyan-400 focus:ring-1 focus:ring-cyan-400 transition-all duration-300" required />
             <label class="absolute left-4 -top-2.5 bg-[#050A28] px-1 text-xs text-cyan-400 transition-all peer-placeholder-shown:text-base peer-placeholder-shown:text-gray-400 peer-placeholder-shown:top-4 peer-focus:-top-2.5 peer-focus:text-cyan-400 peer-focus:text-xs">
              CHAVE DE ACESSO
            </label>
          </div>
        </div>

        <button type="submit"
          class="w-full py-4 px-4 bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 text-white font-bold uppercase tracking-widest rounded-lg transition-all duration-300 transform hover:scale-[1.02] hover:shadow-[0_0_30px_rgba(0,255,255,0.4)] relative overflow-hidden group"
          :disabled="loginApi.showLoading">
          <span class="relative z-10" v-if="!loginApi.showLoading">INICIAR SESSÃO</span>
          <span v-else class="animate-pulse">PROCESSANDO...</span>
          <div class="absolute inset-0 h-full w-full bg-gradient-to-r from-transparent via-white/20 to-transparent -translate-x-full group-hover:animate-shimmer"></div>
        </button>
      </form>
    </section>
  </main>
</template>

<script>
import { useVuelidate } from '@vuelidate/core';
import { required, email } from '@vuelidate/validators';
import { mapGetters } from 'vuex';
import { login } from 'v3/api/auth'; // Import CORRETO para v3
import { useAlert } from 'dashboard/composables';

export default {
  setup() { return { v$: useVuelidate() }; },
  data() { return { credentials: { email: '', password: '' }, loginApi: { showLoading: false } }; },
  computed: { ...mapGetters({ globalConfig: 'globalConfig/get' }) },
  methods: {
    showAlertMessage(message) { this.loginApi.showLoading = false; useAlert(message); },
    submitLogin() {
      this.loginApi.showLoading = true;
      login(this.credentials).then(() => { window.location = '/app'; })
        .catch((error) => { this.showAlertMessage(error?.message || 'Acesso Negado'); });
    },
    submitFormLogin() { this.submitLogin(); },
  },
};
</script>

<style scoped>
.glass-card {
  background: rgba(5, 10, 40, 0.7);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(0, 255, 255, 0.2);
  box-shadow: 0 0 50px rgba(0,0,0,0.5), inset 0 0 20px rgba(0, 255, 255, 0.05);
}
.glow-text { text-shadow: 0 0 15px rgba(0, 255, 255, 0.6); }
.stars-layer {
  width: 100%; height: 100%;
  background-image: radial-gradient(1px 1px at 10% 10%, white 1px, transparent 0), radial-gradient(2px 2px at 50% 50%, white 1px, transparent 0);
  background-size: 400px 400px;
  opacity: 0.2;
}
.scanlines {
  background: linear-gradient(to bottom, rgba(255,255,255,0), rgba(255,255,255,0) 50%, rgba(0,0,0,0.1) 50%, rgba(0,0,0,0.1));
  background-size: 100% 4px;
  position: absolute; inset: 0; pointer-events: none; opacity: 0.3;
}
.animate-entry { animation: slideUp 0.8s cubic-bezier(0.2, 0.8, 0.2, 1); }
@keyframes slideUp { from { opacity: 0; transform: translateY(40px); } to { opacity: 1; transform: translateY(0); } }
@keyframes shimmer { 100% { transform: translateX(100%); } }
.group-hover\:animate-shimmer:hover { animation: shimmer 1s infinite; }
</style>
