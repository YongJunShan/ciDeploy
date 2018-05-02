FROM openjdk:8

# Setup useful environment variables
ENV CONF_HOME     /var/atlassian/confluence
ENV CONF_INSTALL  /opt/atlassian/confluence
ENV CONF_VERSION  6.7.2
ENV TZ		  Asia/Shanghai

ENV JAVA_CACERTS  $JAVA_HOME/jre/lib/security/cacerts
ENV CERTIFICATE   $CONF_HOME/certificate

RUN set -x \
    && cp /usr/share/zoneinfo/$TZ /etc/localtime

COPY "atlassian-confluence-${CONF_VERSION}.tar.gz" "/tmp/atlassian-confluence-${CONF_VERSION}.tar.gz"
COPY "mysql-connector-java-5.1.39-bin.jar" "/tmp/mysql-connector-java-5.1.39-bin.jar"

ADD  sources.list /etc/apt/sources.list

# Install Atlassian Confluence and helper tools and setup initial home
# directory structure.
RUN set -x \
    && apt-get update --quiet \
    && apt-get install --quiet --yes --no-install-recommends libtcnative-1 xmlstarlet bash fonts-dejavu-extra fonts-dejavu-core \
    && apt-get clean \
    && mkdir -p                "${CONF_HOME}" \
    && mkdir -p                "${CONF_INSTALL}/conf" \
    && tar -xf  	       /tmp/atlassian-confluence-${CONF_VERSION}.tar.gz --directory "${CONF_INSTALL}" --strip-components=1 --no-same-owner \
    && mv 		       /tmp/mysql-connector-java-5.1.39-bin.jar "${CONF_INSTALL}/confluence/WEB-INF/lib" \
    && chmod -R 700            "${CONF_INSTALL}/conf" \
    && chmod -R 700            "${CONF_INSTALL}/temp" \
    && chmod -R 700            "${CONF_INSTALL}/logs" \
    && chmod -R 700            "${CONF_INSTALL}/work" \
    && echo -e                 "\nconfluence.home=$CONF_HOME" >> "${CONF_INSTALL}/confluence/WEB-INF/classes/confluence-init.properties" \
    && xmlstarlet              ed --inplace \
        --delete               "Server/@debug" \
        --delete               "Server/Service/Connector/@debug" \
        --delete               "Server/Service/Connector/@useURIValidationHack" \
        --delete               "Server/Service/Connector/@minProcessors" \
        --delete               "Server/Service/Connector/@maxProcessors" \
        --delete               "Server/Service/Engine/@debug" \
        --delete               "Server/Service/Engine/Host/@debug" \
        --delete               "Server/Service/Engine/Host/Context/@debug" \
                               "${CONF_INSTALL}/conf/server.xml" \
    && touch -d "@0"           "${CONF_INSTALL}/conf/server.xml"

RUN set -x \
    && rm -rf /tmp/*

# Use the default unprivileged account. This could be considered bad practice
# on systems where multiple processes end up being executed by 'daemon' but
# here we only ever run one process anyway.
# USER daemon:daemon

# Expose default HTTP connector port.
EXPOSE 8090 8091

# Set volume mount points for installation and home directory. Changes to the
# home directory needs to be persisted as well as parts of the installation
# directory due to eg. logs.
VOLUME ["/var/atlassian/confluence", "/opt/atlassian/confluence/logs"]

# Set the default working directory as the Confluence home directory.
WORKDIR /var/atlassian/confluence

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]

# Run Atlassian Confluence as a foreground process by default.
CMD ["/opt/atlassian/confluence/bin/start-confluence.sh", "-fg"]
