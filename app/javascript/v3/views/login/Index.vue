<script>
// utils and composables
import { login } from '../../api/auth';
import { mapGetters } from 'vuex';
import { useAlert } from 'dashboard/composables';
import { required, email } from '@vuelidate/validators';
import { useVuelidate } from '@vuelidate/core';
import { SESSION_STORAGE_KEYS } from 'dashboard/constants/sessionStorage';
import SessionStorage from 'shared/helpers/sessionStorage';

// components
import SimpleDivider from '../../components/Divider/SimpleDivider.vue';
import GoogleOAuthButton from '../../components/GoogleOauth/Button.vue';
import Spinner from 'shared/components/Spinner.vue';
import Icon from 'dashboard/components-next/icon/Icon.vue';
import MfaVerification from 'dashboard/components/auth/MfaVerification.vue';

const ERROR_MESSAGES = {
  'no-account-found': 'LOGIN.OAUTH.NO_ACCOUNT_FOUND',
  'business-account-only': 'LOGIN.OAUTH.BUSINESS_ACCOUNTS_ONLY',
  'saml-authentication-failed': 'LOGIN.SAML.API.ERROR_MESSAGE',
  'saml-not-enabled': 'LOGIN.SAML.API.ERROR_MESSAGE',
};

const IMPERSONATION_URL_SEARCH_KEY = 'impersonation';

export default {
  components: {
    GoogleOAuthButton,
    Spinner,
    SimpleDivider,
    MfaVerification,
    Icon,
  },
  props: {
    ssoAuthToken: { type: String, default: '' },
    ssoAccountId: { type: String, default: '' },
    ssoConversationId: { type: String, default: '' },
    email: { type: String, default: '' },
    authError: { type: String, default: '' },
  },
  setup() {
    return {
      v$: useVuelidate(),
    };
  },
  data() {
    return {
      credentials: {
        email: '',
        password: '',
      },
      loginApi: {
        message: '',
        showLoading: false,
        hasErrored: false,
      },
      error: '',
      mfaRequired: false,
      mfaToken: null,
    };
  },
  validations() {
    return {
      credentials: {
        password: {
          required,
        },
        email: {
          required,
          email,
        },
      },
    };
  },
  computed: {
    ...mapGetters({ globalConfig: 'globalConfig/get' }),
    allowedLoginMethods() {
      return window.chatwootConfig.allowedLoginMethods || ['email'];
    },
    showGoogleOAuth() {
      return (
        this.allowedLoginMethods.includes('google_oauth') &&
        Boolean(window.chatwootConfig.googleOAuthClientId)
      );
    },
    showSignupLink() {
      return window.chatwootConfig.signupEnabled === 'true';
    },
    showSamlLogin() {
      return this.allowedLoginMethods.includes('saml');
    },
  },
  created() {
    if (this.ssoAuthToken) {
      this.submitLogin();
    }
    if (this.authError) {
      const messageKey = ERROR_MESSAGES[this.authError] ?? 'LOGIN.API.UNAUTH';
      const translatedMessage = this.getTranslatedMessage(messageKey);
      useAlert(translatedMessage);
      this.requestIdleCallbackPolyfill(() => {
        const { query } = this.$route;
        this.$router.replace({ query: { ...query, error: undefined } });
      });
    }
  },
  methods: {
    getTranslatedMessage(key) {
      switch (key) {
        case 'LOGIN.OAUTH.NO_ACCOUNT_FOUND':
          return this.$t('LOGIN.OAUTH.NO_ACCOUNT_FOUND');
        case 'LOGIN.OAUTH.BUSINESS_ACCOUNTS_ONLY':
          return this.$t('LOGIN.OAUTH.BUSINESS_ACCOUNTS_ONLY');
        case 'LOGIN.API.UNAUTH':
        default:
          return this.$t('LOGIN.API.UNAUTH');
      }
    },
    requestIdleCallbackPolyfill(callback) {
      if (window.requestIdleCallback) {
        window.requestIdleCallback(callback);
      } else {
        setTimeout(callback, 0);
      }
    },
    showAlertMessage(message) {
      this.loginApi.showLoading = false;
      this.loginApi.message = message;
      useAlert(this.loginApi.message);
    },
    handleImpersonation() {
      const urlParams = new URLSearchParams(window.location.search);
      const impersonation = urlParams.get(IMPERSONATION_URL_SEARCH_KEY);
      if (impersonation) {
        SessionStorage.set(SESSION_STORAGE_KEYS.IMPERSONATION_USER, true);
      }
    },
    submitLogin() {
      this.loginApi.hasErrored = false;
      this.loginApi.showLoading = true;

      const credentials = {
        email: this.email
          ? decodeURIComponent(this.email)
          : this.credentials.email,
        password: this.credentials.password,
        sso_auth_token: this.ssoAuthToken,
        ssoAccountId: this.ssoAccountId,
        ssoConversationId: this.ssoConversationId,
      };

      login(credentials)
        .then(result => {
          if (result?.mfaRequired) {
            this.loginApi.showLoading = false;
            this.mfaRequired = true;
            this.mfaToken = result.mfaToken;
            return;
          }

          this.handleImpersonation();
          this.showAlertMessage(this.$t('LOGIN.API.SUCCESS_MESSAGE'));
        })
        .catch(response => {
          if (this.email) {
            window.location = '/app/login';
          }
          this.loginApi.hasErrored = true;
          this.showAlertMessage(
            response?.message || this.$t('LOGIN.API.UNAUTH')
          );
        });
    },
    submitFormLogin() {
      if (this.v$.credentials.email.$invalid && !this.email) {
        this.showAlertMessage(this.$t('LOGIN.EMAIL.ERROR'));
        return;
      }
      this.submitLogin();
    },
    handleMfaVerified() {
      this.handleImpersonation();
      window.location = '/app';
    },
    handleMfaCancel() {
      this.mfaRequired = false;
      this.mfaToken = null;
      this.credentials.password = '';
    },
  },
};
</script>

<template>
  <main class="stellar-login-container relative w-full min-h-screen flex items-center justify-center overflow-hidden">
    <!-- Animated Background -->
    <div class="stellar-bg-anim"></div>
    <div class="stellar-stars"></div>

    <div class="relative z-10 w-full max-w-md p-8">
        <!-- Brand Title (Replaces Logo) -->
        <div class="text-center mb-8">
            <h1 class="stellar-brand-title text-4xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 to-blue-600 animate-pulse">
                STELLAR SYNTEC
            </h1>
            <p class="text-cyan-200 mt-2 tracking-widest text-xs uppercase opacity-70">Authorized Access Only</p>
        </div>

      <section v-if="mfaRequired" class="mt-4">
         <div class="stellar-glass-card p-8 rounded-xl">
            <MfaVerification
            :mfa-token="mfaToken"
            @verified="handleMfaVerified"
            @cancel="handleMfaCancel"
            />
        </div>
      </section>

      <section
        v-else
        class="stellar-glass-card p-8 rounded-xl shadow-2xl backdrop-blur-xl"
        :class="{ 'animate-wiggle': loginApi.hasErrored }"
      >
        <div v-if="!email">
          <div class="flex flex-col gap-4 mb-6">
            <GoogleOAuthButton v-if="showGoogleOAuth" class="stellar-btn-secondary" />
            <div v-if="showSamlLogin" class="text-center">
              <router-link
                to="/app/login/sso"
                class="stellar-btn-secondary flex items-center justify-center w-full py-3 rounded-md transition-all duration-300"
              >
                <Icon
                  icon="i-lucide-lock-keyhole"
                  class="size-5 text-cyan-400"
                />
                <span class="ml-2 text-base font-medium text-cyan-100">
                  {{ $t('LOGIN.SAML.LABEL') }}
                </span>
              </router-link>
            </div>
            <SimpleDivider
              v-if="showGoogleOAuth || showSamlLogin"
              :label="$t('COMMON.OR')"
              class="uppercase text-cyan-500/50"
            />
          </div>

          <form class="space-y-6" @submit.prevent="submitFormLogin">
            <div class="stellar-input-group">
                <label class="block text-cyan-300 text-sm font-bold mb-2 uppercase tracking-wide">{{ $t('LOGIN.EMAIL.LABEL') }}</label>
                <input
                    v-model="credentials.email"
                    type="text"
                    required
                    class="stellar-input w-full px-4 py-3 rounded bg-black/30 border border-cyan-900 focus:border-cyan-400 focus:ring-1 focus:ring-cyan-400 focus:outline-none text-white transition-all duration-300 placeholder-cyan-800/50"
                    :placeholder="$t('LOGIN.EMAIL.PLACEHOLDER')"
                    @input="v$.credentials.email.$touch"
                />
            </div>

            <div class="stellar-input-group">
                <label class="block text-cyan-300 text-sm font-bold mb-2 uppercase tracking-wide">{{ $t('LOGIN.PASSWORD.LABEL') }}</label>
                <input
                    v-model="credentials.password"
                    type="password"
                    required
                    class="stellar-input w-full px-4 py-3 rounded bg-black/30 border border-cyan-900 focus:border-cyan-400 focus:ring-1 focus:ring-cyan-400 focus:outline-none text-white transition-all duration-300 placeholder-cyan-800/50"
                    :placeholder="$t('LOGIN.PASSWORD.PLACEHOLDER')"
                    @input="v$.credentials.password.$touch"
                />
                <div class="text-right mt-2" v-if="!globalConfig.disableUserProfileUpdate">
                    <router-link
                        to="auth/reset/password"
                        class="text-xs text-cyan-400 hover:text-cyan-200 transition-colors"
                    >
                        {{ $t('LOGIN.FORGOT_PASSWORD') }}
                    </router-link>
                </div>
            </div>

            <button
                type="submit"
                class="stellar-btn-primary w-full py-3 px-6 rounded font-bold uppercase tracking-widest text-black bg-cyan-400 hover:bg-cyan-300 hover:shadow-[0_0_20px_rgba(0,255,255,0.6)] transition-all duration-300 transform hover:scale-[1.02]"
                :disabled="loginApi.showLoading"
            >
                <span v-if="!loginApi.showLoading">{{ $t('LOGIN.SUBMIT') }}</span>
                <Spinner v-else color-scheme="primary" size="sm" />
            </button>
          </form>

            <p v-if="showSignupLink" class="mt-6 text-sm text-center text-cyan-500/70">
                {{ $t('COMMON.OR') }}
                <router-link to="auth/signup" class="text-cyan-400 hover:text-white transition-colors ml-1 font-semibold">
                {{ $t('LOGIN.CREATE_NEW_ACCOUNT') }}
                </router-link>
            </p>

        </div>
        <div v-else class="flex items-center justify-center">
          <Spinner color-scheme="primary" size="" />
        </div>
      </section>
    </div>
  </main>
</template>

<style lang="scss" scoped>
@import 'dashboard/assets/scss/stellar-effects';

.stellar-login-container {
    background: radial-gradient(circle at center, #080824 0%, #00001A 100%);
    color: white;
}

.stellar-bg-anim {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background:
        radial-gradient(circle at 15% 50%, rgba(0, 255, 255, 0.05) 0%, transparent 25%),
        radial-gradient(circle at 85% 30%, rgba(80, 0, 255, 0.05) 0%, transparent 25%);
    animation: pulseBg 10s infinite alternate;
}

@keyframes pulseBg {
    0% { transform: scale(1); opacity: 0.5; }
    100% { transform: scale(1.1); opacity: 0.8; }
}

.stellar-glass-card {
    @include stellar-glass-panel;
    @include stellar-neon-border;
}

.stellar-brand-title {
    text-shadow: 0 0 10px rgba(0, 255, 255, 0.5);
    font-family: 'Orbitron', sans-serif; /* Assuming font availability or fallback */
    letter-spacing: 2px;
}

.stellar-btn-secondary {
    background: rgba(0, 0, 0, 0.3);
    border: 1px solid rgba(0, 255, 255, 0.2);
    color: #00ffff;
    &:hover {
        background: rgba(0, 255, 255, 0.1);
        border-color: #00ffff;
    }
}
</style>
