# Set the base image
FROM eclipse-temurin:21-jdk

LABEL maintainer="MQTT.Cool Server Development Team <support@mqtt.cool>"

# Set environment variables to identify the right MQTT.Cool version and edition
ENV MQTTCOOL_VERSION=2.1.0
ENV MQTTCOOL_URL_DOWNLOAD=https://lightstreamer.com/mqttcool/server/${MQTTCOOL_VERSION}/mqtt.cool-${MQTTCOOL_VERSION}.tar.gz

# Install the required packages
RUN set -ex; \
        mkdir /mqtt.cool; \
        cd /mqtt.cool; \
        apt-get -y update; \
        apt-get install -y --no-install-recommends gnupg dirmngr curl; \
        rm -rf /var/lib/apt/lists/*; \
# Import Lighstreamer's public key
        gpg --batch --keyserver hkp://keyserver.ubuntu.com --recv-keys 9B90BFD14309C7DA5EF58D7D4A8C08966F29B4D2; \
# Download the distribution from the MQTT.Cool site, verify the signature, and unpack
        curl -fSL -o mqtt.cool.tar.gz ${MQTTCOOL_URL_DOWNLOAD}; \
        curl -fSL -o mqtt.cool.tar.gz.asc ${MQTTCOOL_URL_DOWNLOAD}.asc; \
        gpg --batch --verify mqtt.cool.tar.gz.asc mqtt.cool.tar.gz; \
        tar -xvf mqtt.cool.tar.gz --strip-components=1; \
# Replace the fictitious jdk path with the JAVA_HOME environment variable in the launch script file
        sed -i -- 's/\/usr\/jdk-11/$JAVA_HOME/' bin/unix-like/mc.sh; \
# Addjust logging configurations in order to log only to the standard output
        sed -i -e 's/<appender-ref ref="LSDailyRolling" \/>/<appender-ref ref="LSConsole" \/>/' \
               -e 's/<appender-ref ref="MQTTCoolRolling" \/>/<appender-ref ref="LSConsole" \/>/'  \
               -e '/<logger name="LightstreamerLogger.init/,+2s/<appender-ref ref="LSConsole" \/>/<!-- <appender-ref ref="LSConsole" \/> -->/' \
               -e '/<logger name="LightstreamerLogger.license/,+2s/<appender-ref ref="LSConsole" \/>/<!-- <appender-ref ref="LSConsole" \/> -->/' \
               conf/log_configuration.xml; \
# Add new user and group
        groupadd -r -g 1001 mqttcool; \
        useradd --no-log-init -r -g mqttcool -u 1001 mqttcool; \
# Change ownership of tye mqtt.cool folder
        chown -R mqttcool:mqttcool ../mqtt.cool; \
# Finally, remove no longer needed files
        rm mqtt.cool.tar.gz mqtt.cool.tar.gz.asc; \
        apt-get remove -y gnupg dirmngr curl

USER mqttcool

# Export TCP port 8080
EXPOSE 8080

# Set the final working dir
WORKDIR /mqtt.cool/bin/unix-like

# Start the server
CMD ["./mc.sh", "run"]
