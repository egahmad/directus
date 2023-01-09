ARG NODE_VERSION=18-alpine

FROM node:${NODE_VERSION} as builder

ARG TARGETPLATFORM

WORKDIR /directus
COPY /dist .
RUN \
  if [ "$TARGETPLATFORM" = 'linux/arm64' ]; then \
  apk --no-cache add \
  python3 \
  build-base \
  && ln -sf /usr/bin/python3 /usr/bin/python \
  ; fi
RUN npm install --no-package-lock
RUN rm *.tgz

# Directus image
FROM node:${NODE_VERSION}

ARG VERSION
ARG REPOSITORY=directus/directus

LABEL directus.version="${VERSION}"

################################################
# egahmad changes start here
USER root
WORKDIR /directus

RUN apk update && apk upgrade
RUN apk add git
RUN apk add nano

RUN npm install directus-extension-editorjs
# egahmad changes ends here
################################################

# Default environment variables
# (see https://docs.directus.io/reference/environment-variables/)
ENV \
  DB_CLIENT="sqlite3" \
  DB_FILENAME="/directus/database/database.sqlite" \
  EXTENSIONS_PATH="/directus/extensions" \
  STORAGE_LOCAL_ROOT="/directus/uploads"

RUN \
  # Upgrade system and install 'ssmtp' to be able to send mails
  apk upgrade --no-cache && apk add --no-cache \
  ssmtp \
  # Add support for specifying the timezone of the container
  # using the "TZ" environment variable.
  tzdata \
  # Create directory for Directus with corresponding ownership
  # (can be omitted on newer Docker versions since WORKDIR below will do the same)
  && mkdir /directus && chown node:node /directus


# Switch to user 'node' and directory '/directus'
USER node
WORKDIR /directus

# disable npm update warnings
RUN echo "update-notifier=false" >> ~/.npmrc

COPY --from=builder --chown=node:node /directus .

RUN \
  # Create data directories
  mkdir -p \
    database \
    extensions \
    uploads

# Expose data directories as volumes
VOLUME \
  /directus/database \
  /directus/extensions \
  /directus/uploads
  
# egahmad changes start here
WORKDIR /directus

RUN apk update && apk upgrade
RUN apk add git
RUN apk add nano

# egahmad changes ends here

EXPOSE 8055
CMD npx directus bootstrap && npx directus start
