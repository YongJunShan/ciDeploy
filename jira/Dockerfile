FROM openjdk:8

# Configuration variables.
ENV JIRA_HOME     /var/atlassian/jira
ENV JIRA_INSTALL  /opt/atlassian/jira
ENV JIRA_VERSION  7.3.9
ENV TZ		  Asia/Shanghai

COPY atlassian-jira-software-${JIRA_VERSION}.tar.gz /tmp
COPY mysql-connector-java-5.1.39-bin.jar /tmp
ADD  sources.list /etc/apt/sources.list

RUN set -x \
    && cp /usr/share/zoneinfo/$TZ /etc/localtime

# Install Atlassian JIRA and helper tools and setup initial home
# directory structure.
# && chown -R daemon:daemon  "${JIRA_HOME}" \
RUN set -x \
    && apt-get update --quiet \
    && apt-get install --quiet --yes --no-install-recommends libtcnative-1 xmlstarlet \
    && apt-get clean \
    && mkdir -p                "${JIRA_HOME}" \
    && mkdir -p                "${JIRA_HOME}/caches/indexes" \
    && mkdir -p                "${JIRA_INSTALL}/conf/Catalina" \
    && tar -xf 		       /tmp/atlassian-jira-software-${JIRA_VERSION}.tar.gz --directory "${JIRA_INSTALL}" --strip-components=1 --no-same-owner \
    && cp 		       /tmp/mysql-connector-java-5.1.39-bin.jar "${JIRA_INSTALL}/lib" \
    && chmod -R 700            "${JIRA_INSTALL}/conf" \
    && chmod -R 700            "${JIRA_INSTALL}/logs" \
    && chmod -R 700            "${JIRA_INSTALL}/temp" \
    && chmod -R 700            "${JIRA_INSTALL}/work" \
    && sed --in-place          "s/java version/openjdk version/g" "${JIRA_INSTALL}/bin/check-java.sh" \
    && echo -e                 "\njira.home=$JIRA_HOME" >> "${JIRA_INSTALL}/atlassian-jira/WEB-INF/classes/jira-application.properties" \
    && touch -d "@0"           "${JIRA_INSTALL}/conf/server.xml"

RUN set -x \
    && rm -f /tmp/atlassian-jira-software-${JIRA_VERSION}.tar.gz mysql-connector-java-5.1.39-bin.jar

# Use the default unprivileged account. This could be considered bad practice
# on systems where multiple processes end up being executed by 'daemon' but
# here we only ever run one process anyway.
# USER daemon:daemon

# Expose default HTTP connector port.
EXPOSE 8080

# Set volume mount points for installation and home directory. Changes to the
# home directory needs to be persisted as well as parts of the installation
# directory due to eg. logs.
VOLUME ["/var/atlassian/jira", "/opt/atlassian/jira/logs"]

# Set the default working directory as the installation directory.
WORKDIR /var/atlassian/jira

COPY "docker-entrypoint.sh" "/"
ENTRYPOINT ["/docker-entrypoint.sh"]

# Run Atlassian JIRA as a foreground process by default.
CMD ["/opt/atlassian/jira/bin/catalina.sh", "run"]
