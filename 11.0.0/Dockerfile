#
# Dockerfile to create a Compute Server Node
# Define the build stage
FROM ubuntu:20.04 as build
ARG GRB_VERSION=11.0.0
ARG GRB_SHORT_VERSION=11.0
ARG TARGETPLATFORM

# Create a script to determine GRB_PLATFORM based on TARGETPLATFORM
RUN if [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
        echo "armlinux64" > /platform.txt; \
      else \
        echo "linux64" > /platform.txt; \
    fi

ENV GRB_PLATFORM=${GRB_PLATFORM}

WORKDIR /opt

# Install dependencies and download the appropriate Gurobi package
RUN export SUFFIX=$(cat /platform.txt) && echo $GRB_PLATFORM \
    && apt-get update \
    && apt-get install --no-install-recommends -y \
       ca-certificates \
       wget \
    && update-ca-certificates \
    && wget -v https://packages.gurobi.com/${GRB_SHORT_VERSION}/gurobi_server${GRB_VERSION}_$SUFFIX.tar.gz \
    && tar -xvf gurobi_server${GRB_VERSION}_$SUFFIX*.tar.gz \
    && pwd && ls -alrt \
    && rm -f gurobi_server${GRB_VERSION}_$SUFFIX*.tar.gz \
    && mv -f gurobi_server* gurobi_server \
    && rm -rf gurobi_server/$GRB_PLATFORM/docs \
    && mv -f gurobi_server/$GRB_PLATFORM*  gurobi_server/linux
# After the file renaming, a clean image is built

# Define the final stage
FROM ubuntu:20.04 AS package

ARG GRB_VERSION=11.0.0
ARG TARGETPLATFORM

LABEL vendor="Gurobi"
LABEL version=${GRB_VERSION}

# Update the system and install necessary packages
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
       ca-certificates \
       p7zip-full \
       zip \
    && update-ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt

# Copy files from the build stage
COPY --from=build /opt/gurobi_server/linux/bin/data ./gurobi_server/linux/bin/data
COPY --from=build /opt/gurobi_server/linux/bin/grb_rs ./gurobi_server/linux/bin/grb_rs
COPY --from=build /opt/gurobi_server/linux/bin/grb_rs.cnf ./gurobi_server/linux/bin/grb_rs.cnf
COPY --from=build /opt/gurobi_server/linux/bin/grbunzip ./gurobi_server/linux/bin/grbunzip
COPY --from=build /opt/gurobi_server/linux/bin/grbzip ./gurobi_server/linux/bin/grbzip
COPY --from=build /opt/gurobi_server/linux/bin/grbcluster ./gurobi_server/linux/bin/grbcluster
COPY --from=build /opt/gurobi_server/linux/EULA.pdf ./gurobi_server/linux/EULA.pdf
COPY --from=build /opt/gurobi_server/linux/resources/grb_rs ./gurobi_server/linux/resources/grb_rs


# Set environment variables and permissions
ENV GRB_HOME=/opt/gurobi_server/linux
ENV PATH $GRB_HOME/bin:$PATH
WORKDIR $GRB_HOME/bin

# Changes group permissions to run as a non-root user for better security
RUN chgrp -R 0 /opt/gurobi_server/linux/bin && \
    chmod -R g=u /opt/gurobi_server/linux/bin

# User to run the container
USER 1001

# Expose command line
ENTRYPOINT ["grb_rs", "--port=61000", "--console-ts"]
EXPOSE 61000
