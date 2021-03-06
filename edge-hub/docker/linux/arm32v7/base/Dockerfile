ARG base_tag=2.1.2-runtime-bionic-arm32v7
ARG num_procs=4

FROM arm32v7/buildpack-deps:bionic as builder

RUN apt-get update && \
    apt-get install -y libsnappy-dev

RUN git clone -b v5.4.6 --depth 1 https://github.com/facebook/rocksdb.git && \
    cd rocksdb && \
    CFLAGS="-static-libstdc++ -Wno-psabi -DSNAPPY" PORTABLE=1 make -j ${num_procs} shared_lib && \
    strip librocksdb.so && \
    mkdir -p ../publish && \
    cp -vL librocksdb.so ../publish/

FROM microsoft/dotnet:${base_tag}

# Add an unprivileged user account for running Edge Hub
RUN useradd -ms /bin/bash edgehubuser
ENV EdgeHubUser=edgehubuser

RUN apt-get update && apt-get install -y libcap2-bin libsnappy1v5 && \
    ln -s /lib/arm-linux-gnueabihf/libdl.so.2 /usr/lib/arm-linux-gnueabihf/libdl.so && \
    ln -s /lib/arm-linux-gnueabihf/libc.so.6 /usr/lib/arm-linux-gnueabihf/libc.so && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder publish/* /usr/lib/

# add the CAP_NET_BIND_SERVICE capability to the dotnet binary because
# we are starting edge hub as a non-root user
RUN setcap 'cap_net_bind_service=+ep' /usr/share/dotnet/dotnet