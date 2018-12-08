FROM pstauffer/curl:latest

RUN mkdir /openssl \
    && curl -L https://github.com/openssl/openssl/archive/OpenSSL_1_0_2q.tar.gz | \
      tar -xzC ./openssl --strip-components 1

RUN mkdir /curl \
    && curl -L https://github.com/curl/curl/releases/download/curl-7_56_0/curl-7.56.0.tar.gz | \
      tar -xzC ./curl --strip-components 1

RUN mkdir /git \
    && curl -L https://github.com/git/git/archive/v1.8.0.tar.gz | \
      tar -xzC ./git --strip-components 1

RUN curl -k https://curl.haxx.se/ca/cacert.pem > /cacert.pem



#####################
## Build OpenSSL
#####################

FROM munken/debian:etch-base

RUN apt-get update && \
   apt-get install -y perl curl gcc g++ build-essential

COPY --from=0 /openssl /openssl

RUN mkdir -p /output
WORKDIR /openssl
RUN ./Configure linux-x86_64 --openssldir=/usr/local --prefix=/usr/local shared
RUN make INSTALL_PREFIX=/output install

#####################
## Build curl
#####################

FROM munken/debian:etch-base

RUN apt-get update && \
   apt-get install -y perl curl gcc g++ build-essential

COPY --from=0 /curl /curl
COPY --from=1 /output /

RUN echo "/usr/local/lib" > /etc/ld.so.conf && ldconfig

WORKDIR /curl
RUN ./configure --build=x86_64 --prefix=/usr/local \
    --with-ssl \
    --disable-ldap --disable-ldaps

RUN make
RUN make DESTDIR=/output install


#####################
## Build OpenSSL
#####################

FROM munken/debian:etch-base
COPY --from=0 /git /git

RUN mkdir -p /output

WORKDIR /git

RUN apt-get update 
RUN apt-get install -y libcurl3-dev libz-dev build-essential 
RUN make prefix=/usr NO_OPENSSL=1 NO_TCLTK=1 NO_PYTHON=1 NO_GETTEXT=1 NO_EXPAT=1 DESTDIR=/output install

#####################
## Build final image
#####################

FROM munken/debian:etch-base

COPY --from=1 /output /
COPY --from=2 /output /
COPY --from=3 /output /
RUN echo "/usr/local/lib" > /etc/ld.so.conf && ldconfig

RUN apt-get update && apt-get -y install curl 
COPY --from=0 /cacert.pem /etc/ssl/certs/ca-certificates.crt



    