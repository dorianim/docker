FROM node:14.18.1-bullseye-slim AS builder

RUN apt-get update && apt-get install -y wget curl subversion python3 build-essential
RUN groupadd -g 2000 meteor && useradd -m -u 2001 -g meteor meteor


# download dockerize
ENV DOCKERIZE_VERSION v0.6.1
RUN wget -q https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

USER meteor
ENV METEOR_VERSION 2.5
RUN curl -sL https://install.meteor.com?release=$METEOR_VERSION | sed s/--progress-bar/-sL/g | /bin/sh

ENV TAG v2.4.0
RUN cd ~ \
    && svn checkout https://github.com/bigbluebutton/bigbluebutton/tags/$TAG/bigbluebutton-html5 \
    && mv ~/bigbluebutton-html5 ~/source \
    && cd ~/source \
    && ~/.meteor/meteor npm ci --production \
    && METEOR_DISABLE_OPTIMISTIC_CACHING=1  ~/.meteor/meteor build --architecture os.linux.x86_64 --directory ~/app \
    && rm -rf ~/source

RUN cd ~/app/bundle/programs/server \
    && npm install --production
    
# ------------------------------

FROM node:14.18.1-alpine

RUN addgroup -g 2000 meteor && \
    adduser -D -u 2001 -G meteor meteor && \
    apk add su-exec
COPY --from=builder /usr/local/bin/dockerize /usr/local/bin/dockerize
COPY --from=builder --chown=meteor:meteor /home/meteor/app/bundle /app
COPY entrypoint.sh /entrypoint.sh
COPY bbb-html5.yml /app/bbb-html5.yml.tmpl

ENTRYPOINT ["/entrypoint.sh"]

# lets set the tag again, so that it is include in the image for later version retrieval
ENV TAG v2.4.0
