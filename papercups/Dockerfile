FROM elixir:alpine as builder

LABEL version="1.0"
LABEL maintainer="Pukar Sharma <sharmapukar217@gmail.com>"

ARG MIX_ENV=prod
ARG NODE_ENV=production
ARG APP_VER=0.0.1
ARG USE_IP_V6=false
ARG REQUIRE_DB_SSL=false
ARG AWS_ACCESS_KEY_ID
ARG AWS_SECRET_ACCESS_KEY
ARG BUCKET_NAME
ARG AWS_REGION
ARG PAPERCUPS_STRIPE_SECRET


ENV MIX_ENV=$MIX_ENV
ENV APP_VERSION=$APP_VER
ENV REQUIRE_DB_SSL=$REQUIRE_DB_SSL
ENV USE_IP_V6=$USE_IP_V6
ENV AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
ENV AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
ENV BUCKET_NAME=$BUCKET_NAME
ENV AWS_REGION=$AWS_REGION
ENV PAPERCUPS_STRIPE_SECRET=$PAPERCUPS_STRIPE_SECRET

# install required packages
RUN apk add --no-cache git nodejs yarn python3 ca-certificates gnupg make erlang gcc libc-dev

# setup directories
RUN mkdir -p /app /app/assets
WORKDIR /app

# clone the source code
RUN git clone https://github.com/papercups-io/papercups.git papercups


### Setup client side ###

# fix because of https://github.com/facebook/create-react-app/issues/8413
ENV GENERATE_SOURCEMAP=false

# installing npm packages and building
WORKDIR assets
RUN cp ../papercups/assets/package.json ./ \
    && yarn install --production \
    && yarn cache clean

RUN cp ../papercups/assets/ . -r \
    && yarn cache clean && yarn build && cd ..

WORKDIR /app

# build elixir(backend)
RUN cp papercups/mix.exs papercups/mix.lock papercups/config . -r

RUN mix local.hex --force && \ 
     mix local.rebar --force && \
     mix deps.get --only prod


RUN cp papercups/lib papercups/priv . -r

RUN mix deps.compile \
    && mix phx.digest priv/static


RUN cp papercups/rel . -r
RUN mix release papercups

# deployment
FROM alpine:3.13 AS app
RUN apk add --no-cache openssl ncurses-libs
ENV LANG=C.UTF-8


ENV HOME=/app

RUN adduser -h /app -u 1000 -s /bin/sh -D papercupsuser
COPY --from=builder --chown=papercupsuser:papercupsuser /app/_build/prod/rel/papercups /app
COPY --from=builder --chown=papercupsuser:papercupsuser /app/priv /app/priv

RUN chown -R papercupsuser:papercupsuser /app \
    && cp papercups/docker-entrypoint.sh /entrypoint.sh \
    &&  chmod a+x /entrypoint.sh

USER papercupsuser

EXPOSE 4000
ENTRYPOINT ["/entrypoint.sh"]
CMD ["run"]
