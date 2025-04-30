# pq-openssl-docker

This code snippet can be used to automatically download, compile, and install OpenSSL with [oqs-provider](https://github.com/open-quantum-safe/oqs-provider), enabling post-quantum algorithms provider by OpenQuantumSafe's libOQS library.

```Dockerfile
# BUILDER FOR OPENSSL WITH OQS-PROVIDER
FROM bitnami/minideb:bookworm AS pq-openssl-builder
ENV OPENSSLv3_TAG = 3.4.0
ENV OPENSSLv3_DOWNLOAD_URL = https://github.com/openssl/openssl/releases/download/openssl-$OPENSSLv3_TAG/openssl-$OPENSSLv3_TAG.tar.gz
ENV OQSPROVIDER_TAG = 0.8.0

RUN install_packages \
    wget ca-certificates perl build-essential make git cmake ninja-build

# OPENSSL@3
WORKDIR /root/opensslv3
RUN wget $OPENSSLv3_DOWNLOAD_URL \
    && tar --strip-components=1 -zxvf openssl-$OPENSSLv3_TAG.tar.gz \
    && ./config \
    && make -j $(nproc) \
    && make -j $(nproc) install \
    && ldconfig

# OQS-PROVIDER
WORKDIR /root/oqsprovider
RUN git clone --branch $OQSPROVIDER_TAG --depth 1 https://github.com/open-quantum-safe/oqs-provider.git .\
    && OQSPROV_CMAKE_PARAMS="-DOQS_KEM_ENCODERS=ON" OPENSSL_INSTALL=/usr/local ./scripts/fullbuild.sh

# USE FOLLOWING CODE TO USE IT IN YOUR FINAL IMAGE
# INSTALL PQ-OPENSSL3
# COPY --from=pq-openssl-builder /usr/local/bin/* /usr/local/bin/
# COPY --from=pq-openssl-builder /usr/local/include/* /usr/local/include/
# COPY --from=pq-openssl-builder /usr/local/lib/* /usr/local/lib/
# COPY --from=pq-openssl-builder /usr/local/share/* /usr/local/share/
# COPY --from=pq-openssl-builder /usr/local/ssl/* /usr/local/ssl/

# RUN ldconfig /usr/local/lib

# COPY --from=pq-openssl-builder /root/oqsprovider/_build/lib/oqsprovider.so /usr/local/lib/ossl-modules/oqsprovider.so
# RUN sed -i 's/default = default_sect/default = default_sect\noqsprovider = oqsprovider_sect\n\n\[oqsprovider_sect\]\nactivate = 1/g' /usr/local/ssl/openssl.cnf && sed -i 's/# activate = 1/activate = 1/g' /usr/local/ssl/openssl.cnf
# == END BUILDER FOR OPENSSL WITH OQS-PROVIDER ==
```

Dependency on `bitnami/minideb` image is not mandatory, we chose it as we are using minideb in other projects as well. Feel free to experiment with other images for pq-openssl-builder.
See `Dockerfile` in this repository for example usage.

**Please note that OpenSSL v3.5.0+ introduced support for a subset of PQ algorithms and thus installing oqs-provider might not be necessary anymore in certain use cases. However, oqs-provider might still provide more opportunities for experimentations.**

