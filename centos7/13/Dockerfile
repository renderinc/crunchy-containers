FROM centos:7
ARG POSTGRES_VERSION=13
ARG POSTGRES_FULL_VERSION=13.3
ARG PROTOBUF_VERSION=3.11.4
ARG PROTOBUF_SHA512_SUM=777bbb0e9e2375eaebe6b8c87abd660bac70ee469c9ad00dd25917b82d7fb5bbe33cf87f0d69c90e19d55c07a7285ec20974ba4768623ce9ccfadf147fd5e261
ARG PROTOBUF_C_VERSION=1.3.3
ARG PROTOBUF_C_SHA512_SUM=237b6e6df6ebf4e62a1d402053182f1224c0f35656f30d8fb55ac79945d3d1acf264a7da9f100c1836b90c4acfc1fd96e9a5a95cb47a77d0ddf043aacc99f359
ARG POSTGIS_VERSION=3.1.2
ARG POSTGIS_SHA512_SUM=df9d2fa118987e29acd635912a708e643770ad3df82c098f8129f11ab07a9eafde969302d2d2e59a68fea6fc7312fe768a231dd4cb5de7002c418725da75848a
ARG PG_SIMILARITY_VERSION=1_0
ARG PG_SIMILARITY_SHA512_SUM=919e65d6fa88026a7e994f3e89fd89db55a85abd68de5823a220841a3cd5167d1174a2f009fe126b039beea96b13ac3e29e3ab3f0682607a4995bf6c7921cf06
ARG GEOS_VERSION=39
LABEL \
    name="crunchydata/postgres-gis" \
    vendor="crunchy data" \
    PostgresVersion="${POSTGRES_VERSION}" \
    PostgresFullVersion="${POSTGRES_FULL_VERSION}" \
    Version="7.6" \
    Release="2.2.0" \
    url="https://crunchydata.com" \
    summary="Includes PostGIS extensions on top of crunchy-postgres" \
    description="An identical image of crunchy-postgres with the extra PostGIS packages added for users that require PostGIS." \
    io.k8s.description="postgres-gis container" \
    io.k8s.display-name="Crunchy postgres-gis container" \
    io.openshift.expose-services="" \
    io.openshift.tags="crunchy,database"
ENV PGVERSION="${POSTGRES_VERSION}"
RUN rpm -Uvh https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
# TODO
ADD ./centos7/11/extras/timescale_timescaledb.repo /etc/yum.repos.d/timescale_timescaledb.repo
# TODO: pgaudit13_11 # DONE: hari
# TODO: timescaledb-postgresql-11-1.7.5 # DONE: hari
RUN yum -y update \
    && yum -y install \
        epel-release \
        centos-release-scl \
    && yum -y update glibc-common \
    && yum -y install \
        bind-utils \
        gettext \
        hostname \
        json-c \
        pgaudit15_13 \
        pgbackrest \
        postgresql${POSTGRES_VERSION} \
        postgresql${POSTGRES_VERSION}-contrib \
        postgresql${POSTGRES_VERSION}-server \
        procps-ng \
        timescaledb-2-postgresql-13 \
        openssl
ENV PATH=/usr/pgsql-${POSTGRES_VERSION}/bin:/usr/geos${GEOS_VERSION}/bin:$PATH \
    PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/
RUN yum -y install \
        autoconf \
        automake \
        devtoolset-7 \
        file \
        gcc-c++ \
        gdal-devel \
        geos${GEOS_VERSION}-devel \
        json-c-devel \
        libtool \
        libxml2-devel \
        llvm-toolset-7 \
        llvm5.0 \
        postgresql${POSTGRES_VERSION}-devel \
        proj-devel \
    && echo /usr/local/lib > /etc/ld.so.conf.d/local.conf \
    && mkdir -p /protobuf/ \
    && ( \
        cd /protobuf/ \
        && curl -sSL -o src.tar.gz https://github.com/protocolbuffers/protobuf/archive/v${PROTOBUF_VERSION}.tar.gz \
        && sha512sum -c <<< "${PROTOBUF_SHA512_SUM}  src.tar.gz" \
        && tar --strip-components=1 -xvf src.tar.gz \
        && ./autogen.sh \
        && ./configure \
        && make -j$(nproc) \
        && make install \
        && ldconfig \
    ) \
    && rm -rf /protobuf/ \
    && mkdir -p /protobuf-c/ \
    && ( \
        cd /protobuf-c/ \
        && curl -sSL -o src.tar.gz https://github.com/protobuf-c/protobuf-c/archive/v${PROTOBUF_C_VERSION}.tar.gz \
        && sha512sum -c <<< "${PROTOBUF_C_SHA512_SUM}  src.tar.gz" \
        && tar --strip-components=1 -xvf src.tar.gz \
        && ./autogen.sh \
        && ./configure \
        && make -j$(nproc) \
        && make install \
        && ldconfig \
    ) \
    && rm -rf /protobuf-c/ \
    && mkdir /postgis \
    && ( \
        cd /postgis \
        && curl -sSLO https://download.osgeo.org/postgis/source/postgis-${POSTGIS_VERSION}.tar.gz \
        && sha512sum -c - <<< "${POSTGIS_SHA512_SUM}  postgis-${POSTGIS_VERSION}.tar.gz" \
        && tar -xvf postgis-${POSTGIS_VERSION}.tar.gz \
        && cd postgis-${POSTGIS_VERSION} \
        && ./autogen.sh \
        && ./configure \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -rf /postgis \
    && mkdir -p /pg_similarity/src \
    && ( \
        cd /pg_similarity \
        && curl -sSL -o pg_similarity.tar.gz https://github.com/eulerto/pg_similarity/archive/pg_similarity_${PG_SIMILARITY_VERSION}.tar.gz \
        && sha512sum -c - <<< "${PG_SIMILARITY_SHA512_SUM}  pg_similarity.tar.gz" \
        && tar -xvf pg_similarity.tar.gz --strip-components=1 -C src \
        && cd ./src \
        && USE_PGXS=1 make -j$(nproc) install \
    ) \
    && rm -rf /pg_similarity \
    && yum -y remove \
        autoconf \
        automake \
        devtoolset-7 \
        file \
        gcc-c++ \
        gdal-devel \
        geos${GEOS_VERSION}-devel \
        json-c-devel \
        libtool \
        libxml2-devel \
        llvm-toolset-7 \
        llvm5.0 \
        postgresql${POSTGRES_VERSION}-devel \
        proj-devel \
    && yum -y clean all
ENV PGROOT="/usr/pgsql-${PGVERSION}"
# add path settings for postgres user
# bash_profile is loaded in login, but not with exec
# bashrc to set permissions in OCP when using exec
# HOME is / in OCP
ADD conf/.bash_profile /var/lib/pgsql/
ADD conf/.bashrc /var/lib/pgsql
ADD conf/.bash_profile /
ADD conf/.bashrc /
RUN mkdir -p /opt/cpm/bin /opt/cpm/conf /pgdata /pgwal /pgconf /backup /recover /backrestrepo
RUN chown -R postgres:postgres /opt/cpm /var/lib/pgsql \
    /pgdata /pgwal /pgconf /backup /recover /backrestrepo && \
    chmod -R g=u /opt/cpm /var/lib/pgsql \
    /pgdata /pgwal /pgconf /backup /recover /backrestrepo
# Link pgbackrest.conf to default location for convenience
RUN ln -sf /tmp/pgbackrest.conf /etc/pgbackrest.conf
# add volumes to allow override of pg_hba.conf and postgresql.conf
# add volumes to allow backup of postgres files
# add volumes to offer a restore feature
# add volumes to allow storage of postgres WAL segment files
# add volumes to locate WAL files to recover with
# add volumes for pgbackrest to write to
VOLUME ["/pgconf", "/pgdata", "/pgwal", "/backup", "/recover", "/backrestrepo"]
# open up the postgres port
EXPOSE 5432
ADD bin/postgres /opt/cpm/bin
ADD bin/postgres-gis /opt/cpm/bin
ADD bin/common /opt/cpm/bin
ADD conf/postgres /opt/cpm/conf
# TODO: Version?
# ADD tools/pgmonitor/exporter/postgres /opt/cpm/bin/pgmonitor
RUN chmod g=u /etc/passwd \
    && chmod g=u /etc/group
ENTRYPOINT ["/opt/cpm/bin/uid_postgres.sh"]
USER 26
CMD ["/opt/cpm/bin/start.sh"]
