FROM node:24-alpine as node
FROM ruby:3.4.1-alpine3.21 AS pre-builder

ARG NODE_VERSION="24.13.0"
ARG PNPM_VERSION="10.2.0"
ENV NODE_VERSION=${NODE_VERSION}
ENV PNPM_VERSION=${PNPM_VERSION}
ENV BUNDLER_VERSION=2.5.16
ENV RAILS_ENV=production
ENV NODE_OPTIONS="--max-old-space-size=4096 --openssl-legacy-provider"
ENV BUNDLE_PATH="/gems"

RUN apk update && apk add --no-cache openssl tar build-base tzdata postgresql-dev postgresql-client git curl xz && mkdir -p /var/app && gem install bundler -v "$BUNDLER_VERSION"

COPY --from=node /usr/local/bin/node /usr/local/bin/
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm && ln -s /usr/local/lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx
RUN npm install -g pnpm@latest

WORKDIR /app

# Dependências
COPY chatwoot/Gemfile chatwoot/Gemfile.lock ./
RUN sed -i "s/ruby '.*'/ruby '3.4.1'/g" Gemfile
RUN apk update && apk add --no-cache build-base musl ruby-full ruby-dev gcc make musl-dev openssl openssl-dev g++ linux-headers xz vips
RUN bundle config set --local force_ruby_platform true
RUN bundle config set without 'development test'; bundle install -j 4 -r 3

COPY chatwoot/package.json chatwoot/pnpm-lock.yaml ./
RUN pnpm i

# --- INJEÇÃO DA TRANSMUTAÇÃO ---
COPY chatwoot/ .
# Re-patch Ruby após copia
RUN sed -i "s/ruby '.*'/ruby '3.4.1'/g" Gemfile

# Injetar Estilos Globais
COPY custom/_next-colors.scss /app/app/javascript/dashboard/assets/scss/_next-colors.scss
COPY custom/stellar-effects.scss /app/app/javascript/dashboard/assets/scss/stellar-effects.scss
RUN echo "@import 'stellar-effects';" >> /app/app/javascript/dashboard/assets/scss/app.scss

# Injetar Login
COPY custom/Login.vue /app/app/javascript/v3/views/login/Index.vue

# Injetar Assets
COPY custom/brand-assets/ /app/public/brand-assets/
COPY custom/favicons/ /app/public/

# Substituição Global de Cor (Blue -> Cyan)
RUN grep -rl "#2781F6" /app | xargs sed -i 's/#2781F6/#00FFFF/g'
# -----------------------------

RUN mkdir -p /app/log
RUN SECRET_KEY_BASE=precompile_placeholder RAILS_LOG_TO_STDOUT=enabled bundle exec rake assets:precompile && rm -rf spec node_modules tmp/cache
RUN git rev-parse HEAD > /app/.git_sha && rm -rf .git .gitignore

# Final Stage
FROM ruby:3.4.1-alpine3.21
ARG NODE_OPTIONS="--max-old-space-size=4096"
ENV NODE_OPTIONS ${NODE_OPTIONS}
ENV RAILS_ENV=production
ENV BUNDLE_PATH="/gems"

RUN apk update && apk add --no-cache build-base openssl tzdata postgresql-client imagemagick git vips
COPY --from=node /usr/local/bin/node /usr/local/bin/
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
COPY --from=pre-builder /gems/ /gems/
COPY --from=pre-builder /app /app

WORKDIR /app
EXPOSE 3000
CMD ["docker/entrypoints/rails.sh"]
