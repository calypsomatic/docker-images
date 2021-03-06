### download tranSMART war ###
FROM alpine:3.7 AS transmart-war

ARG version
ARG artifact_url
ARG artifact

ENV ARTIFACT_VERSION ${version:-18.1-SNAPSHOT}
ENV ARTIFACT_URL ${artifact_url:-http://ec2-35-170-59-132.compute-1.amazonaws.com:8080/artifactory/wars/transmart/transmart}
ENV ARTIFACT ${artifact:-transmart.war}

RUN apk upgrade --update --no-cache \
    && DEPENDENCIES="wget" \
    && apk --no-cache --update add $DEPENDENCIES \
    #
    # get transmart war file
    # wget will recursively search for .war files and save them to /transmartWar directory - Andre
    #
    && wget -r -P /transmartWar -nd -nc -v -L --progress=bar:force -A war ${ARTIFACT_URL}/${ARTIFACT_VERSION}/ \
    && apk del $DEPENDENCIES && rm -rf /var/cache/apk/*

### tranSMART Production ###
FROM tomcat:8.0-jre7-alpine AS production

    # use bash instead of sh. easier for container debugging
RUN rm /bin/sh && ln -s /bin/bash /bin/sh \
    # see: https://github.com/docker-library/openjdk/issues/73
    # see: https://hub.docker.com/r/neduekwunife/openjdk8-jre-alpine-with-fontconfig/
    # add font ttf-dejavu
    && apk add --update ttf-dejavu \
    && rm -rf /var/cache/apk/* \
    #
    # remove default webapps
    && rm -rf /usr/local/tomcat/webapps/examples \
    && rm -rf /usr/local/tomcat/webapps/docs \
    #
    # create configuration directory
    && mkdir -p /root/.grails/transmartConfig

# dockerfile author
LABEL maintainer="andre_rosa@hms.harvard.edu"

# tomcat variables
ENV CATALINA_OPTS "$CATALINA_OPTS -Xms512m -Xmx2g -Djava.awt.headless=true -XX:+UseConcMarkSweepGC"
# required for ttf-dejavu install
ENV LANG en_US.UTF-8

# i2b2/transmart database variables
ENV DB_HOST localhost
ENV DB_USER biomart_user
ENV BIOMART_USER demouser
ENV DB_PORT 1521
ENV DB_DB ORCLPDB1

# email variables
ENV EMAIL_USER example@gmail.com
ENV EMAIL_PASS email_pass
ENV NOTIFICATION_EMAILS example@gmail.com

# guest login
ENV GUEST_AUTO_LOGIN false
ENV GUEST_USER guest

### Available Plugins ###

## JWT Token authentication
ENV CLIENT_ID client_id
ENV CLIENT_SECRET client_secret

## Rserve
ENV RSERVE_EXTERNAL true
ENV RSERVE_HOST rserve
# Rscripts must be an *absolute* path
ENV RSCRIPTS_DIR /transmart-data/R/Rscripts

## Auth0 (disabled by default)
ENV AUTH0_ACTIVE false
ENV AUTH0_DOMAIN domain

## Fractalis (disabled by default)
ENV FRACTALIS_ACTIVE false
ENV FRACTALIS_NODE /fractalis
ENV FRACTALIS_DATA_SOURCE localhost
ENV FRACTALIS_RESOURCE_NAME resource/Demo


# transmart.war
COPY --from=transmart-war /transmartWar/*.war $CATALINA_HOME/webapps/transmart.war

# configuration files
COPY DataSource.groovy /root/.grails/transmartConfig
COPY Config.groovy /root/.grails/transmartConfig
# With nginx as our front-end, make sure scheme protocol and IP is forwarded to tomcat - Andre
COPY server.xml $CATALINA_HOME/conf/

# temporary hack: disable HTTP request validation -Andre
# see https://hms-dbmi.atlassian.net/browse/I2B2TM-24
RUN sed -i 's/#\(tomcat\.util\.http\.parser\.HttpParser\.requestTargetAllow.*$\)/\1/g' conf/catalina.properties


ENTRYPOINT ["./bin/catalina.sh", "run"]
