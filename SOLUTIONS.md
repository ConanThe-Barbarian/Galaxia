# Custom Chatwoot Build - Stellar Syntec Edition (Phase 3: Aesthetic Transmutation)

## 1. Directory Structure

Ensure your `chatwoot-custom` folder is set up as follows:

```
chatwoot-custom/
├── chatwoot/                  # Clone of the official repository
├── custom/                    # Your customization files
│   ├── _next-colors.scss      # Deep Space Palette
│   ├── stellar-effects.scss   # Glassmorphism & Animations
│   ├── Login.vue              # New Futuristic Login
│   ├── en.yml                 # Locale overrides
│   ├── brand-assets/          # Logos
│   └── favicons/              # Favicons
├── Dockerfile                 # The custom Dockerfile
└── docker-compose.yaml        # For Portainer
```

## 2. Code Modifications

### `custom/_next-colors.scss`
*Redefining the visual foundation for Deep Space.*

```scss
@layer base {
  :root {
    // Stellar Syntec Palette: Deep Space & Neon Cyan
    
    // Base Slate (Deep Space Gradients & Backgrounds)
    --slate-1: 0 0 26;      // #00001A (Deep Space)
    --slate-2: 5 5 35;      // #050523 (Slightly Lighter Space)
    --slate-3: 10 10 45;    // Panel Backgrounds
    --slate-4: 15 15 55;
    --slate-5: 20 20 65;
    --slate-6: 30 30 80;
    --slate-7: 50 50 100;
    --slate-8: 70 70 120;
    --slate-9: 100 100 150;
    --slate-10: 130 130 180;
    --slate-11: 0 255 255;  // Text Primary (Cyan for high tech feel)
    --slate-12: 255 255 255; // Text High Contrast

    // Brand Colors (Cyan)
    --blue-1: 0 20 20;
    --blue-2: 0 30 30;
    --blue-3: 0 40 40;
    --blue-4: 0 50 50;
    --blue-5: 0 60 60;
    --blue-6: 0 80 80;
    --blue-7: 0 120 120;
    --blue-8: 0 180 180;
    --blue-9: 0 255 255;    // Stellar Cyan
    --blue-10: 20 255 255;
    --blue-11: 50 255 255;
    --blue-12: 200 255 255;

    // Secondary Accents (Purple)
    --iris-1: 10 0 25;
    --iris-2: 20 0 40;
    --iris-9: 138 43 226;   // Blue Violet

    // Semantic Mapping
    --background-color: 0 0 26;
    --text-blue: 0 255 255;
    --border-container: 0 255 255; // Cyan borders (low opacity handled in effects)
    --border-strong: 0 255 255;
    --border-weak: 10 10 45;

    // Glassmorphism Bases
    --solid-1: 5 5 35;
    --solid-2: 10 10 35;     // Sidebar / Panels
    --solid-3: 15 15 55;
    --solid-active: 0 40 40; // Active states
    
    --solid-blue: 0 40 40;
    --alpha-1: 0, 0, 0, 0.3;
    --alpha-2: 0, 255, 255, 0.05; // Hover tint
    --alpha-3: 0, 0, 26, 0.9;
    --black-alpha-1: 0, 0, 0, 0.6;
    --white-alpha: 255, 255, 255, 0.1;
  }

  .dark {
     @extend :root;
  }
}
```

### `custom/stellar-effects.scss`
*The new file for Glassmorphism, Neon Glows, and Animations.*

```scss
// --- Global Deep Space Background ---
#app, body, html {
  background: linear-gradient(180deg, #00001A 0%, #080824 100%) !important;
  color: #fff;
}

// --- Glassmorphism Mixin ---
@mixin glass-panel {
  background: rgba(10, 10, 35, 0.7) !important;
  backdrop-filter: blur(16px) saturate(180%) !important;
  -webkit-backdrop-filter: blur(16px) saturate(180%) !important;
  border: 1px solid rgba(0, 255, 255, 0.1);
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
}

@mixin neon-glow {
  box-shadow: 0 0 10px rgba(0, 255, 255, 0.4), inset 0 0 5px rgba(0, 255, 255, 0.1);
  border-color: rgba(0, 255, 255, 0.6) !important;
}

// --- Component Overrides ---

// 1. Sidebar High-Tech Look
aside {
  @include glass-panel;
  border-right: 1px solid rgba(0, 255, 255, 0.1) !important;

  // Active Icon Glow
  .active {
    @include neon-glow;
    background: rgba(0, 255, 255, 0.1) !important;
    
    svg {
      filter: drop-shadow(0 0 3px #00FFFF);
      color: #00FFFF !important;
    }
    
    span {
      color: #fff !important;
      text-shadow: 0 0 5px rgba(0, 255, 255, 0.5);
    }
  }

  // Hover Effects
  a:hover {
    background: rgba(0, 255, 255, 0.05);
    span { color: #00FFFF; }
  }
}

// 2. Conversation List & Cards
.conversation--list {
  @include glass-panel;
}

.conversation-card {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border: 1px solid transparent;

  &:hover {
    transform: scale(1.02);
    background: rgba(0, 255, 255, 0.05);
    border-color: rgba(0, 255, 255, 0.3);
  }

  &.active {
    @include neon-glow;
    transform: scale(1.02);
  }
}

// 3. Chat Area
.conversation-panel {
  @include glass-panel;
}

// Bubbles Fade In Up
.message-content {
  animation: fadeInUp 0.4s ease-out forwards;
}

// --- Animations ---
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

// Page Transitions
.slide-enter-active,
.slide-leave-active {
  transition: all 0.3s ease;
}
.slide-enter-from,
.slide-leave-to {
  transform: translateX(20px);
  opacity: 0;
}
```

### `custom/Login.vue`
*Futuristic Login Portal (Replaces `app/javascript/v3/views/login/Index.vue`).*

```vue
<template>
  <main class="flex items-center justify-center w-full min-h-screen relative overflow-hidden bg-slate-900">
    <!-- Animated Deep Space Background (CSS Only) -->
    <div class="absolute inset-0 z-0 bg-gradient-to-b from-[#00001A] to-[#080824]">
      <div class="stars-container"></div>
    </div>

    <!-- Glass Card -->
    <section class="relative z-10 w-full max-w-md p-8 m-4 space-y-6 glass-card animate-fade-in-up">
      <!-- Logo Area -->
      <div class="flex flex-col items-center justify-center mb-8">
        <img
          :src="globalConfig.logo"
          :alt="globalConfig.installationName"
          class="w-auto h-12 mb-4 drop-shadow-glow"
        />
        <h2 class="text-3xl font-bold text-center text-white tracking-widest uppercase glow-text">
          {{ globalConfig.installationName }}
        </h2>
        <p class="mt-2 text-sm text-center text-cyan-200 opacity-80">
          Secure Access Portal
        </p>
      </div>

      <!-- Login Form -->
      <form class="space-y-6" @submit.prevent="submitFormLogin">
        <div class="space-y-4">
          <div class="group">
            <label class="block text-xs font-medium text-cyan-400 uppercase tracking-wider mb-1">Email Identity</label>
            <input
              v-model="credentials.email"
              type="text"
              placeholder="Enter your access ID"
              class="w-full px-4 py-3 bg-black/30 border border-cyan-900/50 rounded text-white placeholder-cyan-900/50 focus:outline-none focus:border-cyan-400 focus:ring-1 focus:ring-cyan-400 transition-all duration-300"
              required
            />
          </div>

          <div class="group">
             <label class="block text-xs font-medium text-cyan-400 uppercase tracking-wider mb-1">Passkey</label>
             <input
              v-model="credentials.password"
              type="password"
              placeholder="••••••••"
              class="w-full px-4 py-3 bg-black/30 border border-cyan-900/50 rounded text-white placeholder-cyan-900/50 focus:outline-none focus:border-cyan-400 focus:ring-1 focus:ring-cyan-400 transition-all duration-300"
              required
            />
          </div>
        </div>

        <button
          type="submit"
          class="w-full py-3 px-4 bg-cyan-600 hover:bg-cyan-500 text-white font-bold uppercase tracking-widest rounded transition-all duration-300 transform hover:scale-105 hover:shadow-[0_0_20px_rgba(0,255,255,0.4)] relative overflow-hidden"
          :disabled="loginApi.showLoading"
        >
          <span v-if="!loginApi.showLoading">Initialize Session</span>
          <span v-else class="animate-pulse">Authenticating...</span>
        </button>
      </form>
    </section>
  </main>
</template>

<script>
import { useVuelidate } from '@vuelidate/core';
import { required, email } from '@vuelidate/validators';
import { mapGetters } from 'vuex';
import { login } from 'dashboard/api/auth';
import { useAlert } from 'dashboard/composables';

export default {
  setup() {
    return { v$: useVuelidate() };
  },
  data() {
    return {
      credentials: { email: '', password: '' },
      loginApi: { showLoading: false, message: '' },
    };
  },
  computed: {
    ...mapGetters({ globalConfig: 'globalConfig/get' }),
  },
  methods: {
    showAlertMessage(message) {
      this.loginApi.showLoading = false;
      useAlert(message);
    },
    submitLogin() {
      this.loginApi.showLoading = true;
      login(this.credentials)
        .then(() => {
          window.location = '/app';
        })
        .catch((error) => {
          this.showAlertMessage(error?.message || 'Authentication Failed');
        });
    },
    submitFormLogin() {
      this.submitLogin();
    },
  },
};
</script>

<style scoped>
/* Glass Card */
.glass-card {
  background: rgba(0, 20, 40, 0.6);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(0, 255, 255, 0.1);
  border-radius: 16px;
  box-shadow: 0 0 40px rgba(0, 0, 0, 0.5), inset 0 0 20px rgba(0, 255, 255, 0.05);
}

.drop-shadow-glow {
  filter: drop-shadow(0 0 10px rgba(0, 255, 255, 0.5));
}

.glow-text {
  text-shadow: 0 0 10px rgba(0, 255, 255, 0.5);
}

/* Subtle Animated Stars Effect */
.stars-container {
  width: 100%;
  height: 100%;
  background-image: 
    radial-gradient(1px 1px at 10% 10%, white 1px, transparent 0),
    radial-gradient(1px 1px at 20% 40%, white 1px, transparent 0),
    radial-gradient(2px 2px at 40% 60%, white 1px, transparent 0),
    radial-gradient(1px 1px at 60% 30%, white 1px, transparent 0),
    radial-gradient(1px 1px at 80% 80%, white 1px, transparent 0);
  background-size: 550px 550px;
  opacity: 0.3;
  animation: moveStars 100s linear infinite;
}

@keyframes moveStars {
  from { background-position: 0 0; }
  to { background-position: 550px 550px; }
}

.animate-fade-in-up {
  animation: fadeInUp 0.8s ease-out;
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>
```

## 3. Dockerfile

```dockerfile
# pre-build stage
# Using Node 22 (LTS) and Ruby 3.4.1 (Stable)
FROM node:22-alpine as node
FROM ruby:3.4.1-alpine3.21 AS pre-builder

ARG NODE_VERSION="22.12.0"
ARG PNPM_VERSION="9.15.0"
ENV NODE_VERSION=${NODE_VERSION}
ENV PNPM_VERSION=${PNPM_VERSION}

ARG BUNDLE_WITHOUT="development:test"
ENV BUNDLE_WITHOUT ${BUNDLE_WITHOUT}
ENV BUNDLER_VERSION=2.5.16

ARG RAILS_SERVE_STATIC_FILES=true
ENV RAILS_SERVE_STATIC_FILES ${RAILS_SERVE_STATIC_FILES}

ARG RAILS_ENV=production
ENV RAILS_ENV ${RAILS_ENV}

# Build Optimization for Memory
ARG NODE_OPTIONS="--max-old-space-size=4096 --openssl-legacy-provider"
ENV NODE_OPTIONS ${NODE_OPTIONS}

ENV BUNDLE_PATH="/gems"

RUN apk update && apk add --no-cache \
  openssl tar build-base tzdata postgresql-dev postgresql-client git curl xz \
  && mkdir -p /var/app \
  && gem install bundler -v "$BUNDLER_VERSION"

COPY --from=node /usr/local/bin/node /usr/local/bin/
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
  && ln -s /usr/local/lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx

RUN npm install -g pnpm@${PNPM_VERSION}

WORKDIR /app

# Copy dependency definitions
COPY chatwoot/Gemfile chatwoot/Gemfile.lock ./

# Patch Gemfile to match Ruby version
RUN sed -i "s/ruby '.*'/ruby '3.4.1'/g" Gemfile

RUN apk update && apk add --no-cache build-base musl ruby-full ruby-dev gcc make musl-dev openssl openssl-dev g++ linux-headers xz vips
RUN bundle config set --local force_ruby_platform true

RUN bundle config set without 'development test'; bundle install -j 4 -r 3

COPY chatwoot/package.json chatwoot/pnpm-lock.yaml ./
RUN pnpm i

# --- CUSTOMIZATION INJECTION START ---
# Copy the original source
COPY chatwoot/ .

# Inject Custom Colors & Effects
COPY custom/_next-colors.scss /app/app/javascript/dashboard/assets/scss/_next-colors.scss
COPY custom/stellar-effects.scss /app/app/javascript/dashboard/assets/scss/stellar-effects.scss

# Register Effects
RUN echo "@import 'stellar-effects';" >> /app/app/javascript/dashboard/assets/scss/app.scss

# Inject Vue Components
# Replacing the Login page (Mapped to the correct location in v3)
COPY custom/Login.vue /app/app/javascript/v3/views/login/Index.vue

# Inject Assets
COPY custom/brand-assets/ /app/public/brand-assets/
COPY custom/favicons/ /app/public/

# Global Color Replacement (Blue -> Cyan)
RUN grep -rl "#2781F6" /app | xargs sed -i 's/#2781F6/#00FFFF/g'

# --- CUSTOMIZATION INJECTION END ---

RUN mkdir -p /app/log

# Compile Assets
RUN SECRET_KEY_BASE=precompile_placeholder RAILS_LOG_TO_STDOUT=enabled bundle exec rake assets:precompile \
  && rm -rf spec node_modules tmp/cache

RUN git rev-parse HEAD > /app/.git_sha
RUN rm -rf .git .gitignore

# final build stage
FROM ruby:3.4.1-alpine3.21

# ... (Standard arguments) ...
ARG NODE_OPTIONS="--max-old-space-size=4096"
ENV NODE_OPTIONS ${NODE_OPTIONS}
ENV RAILS_ENV=production
ENV BUNDLE_PATH="/gems"

RUN apk update && apk add --no-cache \
  build-base openssl tzdata postgresql-client imagemagick git vips

COPY --from=node /usr/local/bin/node /usr/local/bin/
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules

COPY --from=pre-builder /gems/ /gems/
COPY --from=pre-builder /app /app

WORKDIR /app
EXPOSE 3000
CMD ["docker/entrypoints/rails.sh"]
```

## 4. docker-compose.yaml

```yaml
version: '3'

services:
  base: &base
    image: stellar-chatwoot:latest
    env_file: .env
    environment:
      - RAILS_ENV=production
      - NODE_ENV=production
      - INSTALLATION_NAME=Stellar Syntec
      - NODE_OPTIONS=--max-old-space-size=4096

  web:
    <<: *base
    ports:
      - "3000:3000"
    command: bundle exec rails s -p 3000 -b 0.0.0.0
    depends_on:
      - postgres
      - redis
    volumes:
      - storage_data:/app/storage

  worker:
    <<: *base
    command: bundle exec sidekiq -C config/sidekiq.yml
    depends_on:
      - postgres
      - redis
    volumes:
      - storage_data:/app/storage

  postgres:
    image: pgvector/pgvector:pg16
    restart: always
    ports:
      - '5432:5432'
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=chatwoot_production
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres

  redis:
    image: redis:alpine
    restart: always
    volumes:
      - redis_data:/data
    env_file: .env
    environment:
      - REDIS_PASSWORD=${REDIS_PASSWORD}
    command: ["sh", "-c", "redis-server --requirepass \"$REDIS_PASSWORD\""]

volumes:
  postgres_data:
  redis_data:
  storage_data:
```
