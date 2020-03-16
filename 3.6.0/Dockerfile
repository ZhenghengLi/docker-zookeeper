FROM openjdk:8-jre-slim

ARG ZK_VERSION=3.6.0
ARG DISTRO_NAME=apache-zookeeper-${ZK_VERSION}-bin
ARG DISTRO_URL=https://archive.apache.org/dist/zookeeper/zookeeper-${ZK_VERSION}/${DISTRO_NAME}.tar.gz

ENV ZOO_CONF_DIR=/zookeeper/conf \
    ZOO_DATA_DIR=/zookeeper/data \
    ZOO_DATA_LOG_DIR=/zookeeper/datalog \
    ZOO_LOG_DIR=/zookeeper/logs \
    ZOO_TICK_TIME=2000 \
    ZOO_INIT_LIMIT=5 \
    ZOO_SYNC_LIMIT=2 \
    ZOO_AUTOPURGE_PURGEINTERVAL=0 \
    ZOO_AUTOPURGE_SNAPRETAINCOUNT=3 \
    ZOO_MAX_CLIENT_CNXNS=60 \
    ZOO_STANDALONE_ENABLED=true \
    ZOO_ADMINSERVER_ENABLED=true \
    ZOO_LISTEN_ALL=true

RUN set -x \
# install dependencies
&& apt-get update \
&& apt-get install -y --no-install-recommends ca-certificates gosu gnupg netcat wget \
# verify that gosu binary works
&& gosu nobody true \
# download
&& mkdir -pv /opt/tmp\
&& cd /opt/tmp \
&& wget -q "${DISTRO_URL}" \
&& wget -q "${DISTRO_URL}.asc" \
&& wget -q "${DISTRO_URL}.sha512" \
&& sha512sum -c ${DISTRO_NAME}.tar.gz.sha512 \
&& wget https://dist.apache.org/repos/dist/release/zookeeper/KEYS \
&& gpg --import KEYS \
&& gpg --batch --verify "${DISTRO_NAME}.tar.gz.asc" "${DISTRO_NAME}.tar.gz" \
&& tar -xzf "${DISTRO_NAME}.tar.gz" \
&& mv -v "${DISTRO_NAME}" .. \
&& cd .. \
&& rm -rf tmp \
# clean
&& apt-get autoremove -y wget gnupg \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/* \
# set work directories
&& mkdir -pv "${ZOO_CONF_DIR}" "${ZOO_DATA_DIR}" "${ZOO_DATA_LOG_DIR}" "${ZOO_LOG_DIR}" \
&& mv -v /opt/"${DISTRO_NAME}"/conf/* "${ZOO_CONF_DIR}"/

COPY entrypoint.sh /scripts/

## add default work user
RUN set -x \
&& groupadd zookeeper --gid=1000 \
&& useradd -m -g zookeeper --uid=1000 zookeeper \
&& chown zookeeper:zookeeper "${ZOO_DATA_LOG_DIR}" "${ZOO_DATA_DIR}" "${ZOO_LOG_DIR}" \
&& chmod 777 "${ZOO_CONF_DIR}"

ENV PATH=/opt/"${DISTRO_NAME}"/bin:$PATH \
ZOOCFGDIR="${ZOO_CONF_DIR}"

USER zookeeper

ENTRYPOINT [ "/bin/bash", "/scripts/entrypoint.sh" ]
CMD ["bash"]
