# Custom Chatwoot Build - Stellar Syntec Edition

## 1. Directory Structure

Create a new folder `chatwoot-custom` and set it up as follows:

```
chatwoot-custom/
├── chatwoot/                  # Clone of the official repository (git clone https://github.com/chatwoot/chatwoot.git)
├── custom/                    # Your customization files
│   ├── _next-colors.scss      # Dashboard colors
│   ├── woot.scss              # Widget colors
│   ├── colors.js              # Tailwind brand color override
│   ├── en.yml                 # Locale overrides
│   ├── brand-assets/          # Logos
│   │   ├── logo.svg
│   │   └── logo_dark.svg
│   └── favicons/              # Favicons
│       ├── favicon-32x32.png
│       └── ...
├── Dockerfile                 # The custom Dockerfile
└── docker-compose.yaml        # For Portainer
```

## 2. Code Modifications (The `custom/` folder)

### `custom/_next-colors.scss`
*Overrides `app/javascript/dashboard/assets/scss/_next-colors.scss`*

```scss
@layer base {
  :root {
    // Stellar Syntec Palette
    // Backgrounds: Deep Space Blue/Black (RGB: 0, 0, 26)
    // Primary: Cyan (RGB: 0, 255, 255)
    // Secondary: Purple (RGB: 89, 0, 224)

    // SLATE (Backgrounds & Grays)
    --slate-1: 0 0 26;      // Main App Background
    --slate-2: 5 5 35;      // Secondary Background
    --slate-3: 10 10 45;    // Borders / Dividers
    --slate-4: 15 15 55;
    --slate-5: 20 20 65;
    --slate-6: 30 30 80;
    --slate-7: 50 50 100;   // Muted Text
    --slate-8: 70 70 120;
    --slate-9: 100 100 150;
    --slate-10: 130 130 180;
    --slate-11: 180 180 220; // Primary Text
    --slate-12: 250 250 250; // High Contrast Text

    // BLUE (Primary Accents - Cyan)
    --blue-1: 0 20 20;
    --blue-2: 0 30 30;
    --blue-3: 0 40 40;
    --blue-4: 0 50 50;
    --blue-5: 0 60 60;
    --blue-6: 0 80 80;
    --blue-7: 0 120 120;
    --blue-8: 0 180 180;
    --blue-9: 0 255 255;    // Primary Brand Color
    --blue-10: 20 255 255;
    --blue-11: 50 255 255;
    --blue-12: 200 255 255;

    // IRIS (Secondary Accents - Purple)
    --iris-1: 20 0 50;
    --iris-2: 30 0 70;
    --iris-3: 40 0 90;
    --iris-4: 50 0 110;
    --iris-5: 60 0 130;
    --iris-6: 70 0 150;
    --iris-7: 80 0 180;
    --iris-8: 89 0 224;     // Secondary Brand Color
    --iris-9: 89 0 224;
    --iris-10: 100 20 240;
    --iris-11: 120 40 255;
    --iris-12: 230 200 255;

    // Semantic Mapping
    --background-color: 0 0 26;
    --text-blue: 0 255 255;
    --border-container: 10 10 45;
    --border-strong: 15 15 55;
    --border-weak: 10 10 45;

    --solid-1: 5 5 35;
    --solid-2: 10 10 45;     // Cards / Modals
    --solid-3: 15 15 55;
    --solid-active: 20 20 65;
    
    --solid-blue: 0 40 40;
    --solid-iris: 30 0 70;

    --alpha-1: 0, 0, 0, 0.2;
    --alpha-2: 255, 255, 255, 0.05; // Light hover effect
    --alpha-3: 255, 255, 255, 0.9;
    --black-alpha-1: 0, 0, 0, 0.5;
    --black-alpha-2: 0, 0, 0, 0.3;
    --border-blue: 0, 255, 255, 0.5;
    --white-alpha: 255, 255, 255, 0.1;

    // Keep standard grays for safety or map to slate
    --gray-1: 10 10 10;
    --gray-2: 15 15 15;
    --gray-3: 20 20 20;
    --gray-4: 25 25 25;
    --gray-5: 30 30 30;
    --gray-6: 40 40 40;
    --gray-7: 50 50 50;
    --gray-8: 60 60 60;
    --gray-9: 80 80 80;
    --gray-10: 100 100 100;
    --gray-11: 150 150 150;
    --gray-12: 200 200 200;
  }

  // Force Dark Theme consistency
  .dark {
     @extend :root;
  }
}
```

### `custom/woot.scss`
*Overrides `app/javascript/widget/assets/scss/woot.scss`*
(Same content as above).

### `custom/colors.js`
*Overrides `theme/colors.js`*

```javascript
// This file is used to instruct the user.
// In the Dockerfile, we will use sed to patch the existing file to avoid missing imports.
```

### `custom/en.yml`
*Partial override for `config/locales/en.yml`*

```yaml
en:
  chatwoot: "Stellar Syntec"
  brand_name: "Stellar Syntec"
  login:
    title: "Welcome to Stellar Syntec"
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

# Inject Custom Colors
COPY custom/_next-colors.scss /app/app/javascript/dashboard/assets/scss/_next-colors.scss
COPY custom/woot.scss /app/app/javascript/widget/assets/scss/woot.scss

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
