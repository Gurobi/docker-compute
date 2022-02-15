#
# Docker file to create a Compute Server Node
#

FROM ubuntu:20.04 as build
ARG GRB_VERSION=9.5.1
ARG GRB_SHORT_VERSION=9.5

# install server package and copy the files
WORKDIR /opt
RUN apt-get update \
    && apt-get install --no-install-recommends -y\
       ca-certificates  \
       wget \
    && update-ca-certificates \
    && wget -v https://packages.gurobi.com/${GRB_SHORT_VERSION}/gurobi_server${GRB_VERSION}_linux64.tar.gz \
    && tar -xvf gurobi_server${GRB_VERSION}_linux64.tar.gz  \
    && rm -f gurobi_server${GRB_VERSION}_linux64.tar.gz \
    && mv -f gurobi_server* gurobi_server \
    && rm -rf gurobi/linux64/docs
# After the file renaming, a clean image is built

FROM ubuntu:20.04 AS package

ARG GRB_VERSION=9.5.1

LABEL vendor="Gurobi"
LABEL version=${GRB_VERSION}

# update system and certificates
RUN apt-get update \
    && apt-get install --no-install-recommends -y\
       ca-certificates  \
       p7zip-full \
       zip \
    && update-ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt

COPY --from=build /opt/gurobi_server/linux64/bin/data ./gurobi_server/linux64/bin/data
COPY --from=build /opt/gurobi_server/linux64/bin/grb_rs ./gurobi_server/linux64/bin/grb_rs
COPY --from=build /opt/gurobi_server/linux64/bin/grb_rs.cnf ./gurobi_server/linux64/bin/grb_rs.cnf
COPY --from=build /opt/gurobi_server/linux64/bin/grbunzip ./gurobi_server/linux64/bin/grbunzip
COPY --from=build /opt/gurobi_server/linux64/bin/grbzip ./gurobi_server/linux64/bin/grbzip
COPY --from=build /opt/gurobi_server/linux64/bin/grbcluster ./gurobi_server/linux64/bin/grbcluster
COPY --from=build /opt/gurobi_server/linux64/EULA.pdf ./gurobi_server/linux64/EULA.pdf
COPY --from=build /opt/gurobi_server/linux64/resources/grb_rs ./gurobi_server/linux64/resources/grb_rs

ENV GRB_HOME=/opt/gurobi_server/linux64
ENV PATH $GRB_HOME/bin:$PATH
WORKDIR $GRB_HOME/bin
# changes group permissions to run as a non-root user for better security
RUN chgrp -R 0 /opt/gurobi_server/linux64/bin && \
    chmod -R g=u /opt/gurobi_server/linux64/bin
# User to run the container
USER 1001
# expose command line
ENTRYPOINT ["grb_rs", "--port=61000","--console-ts"]
EXPOSE 61000

