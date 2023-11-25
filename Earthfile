VERSION --pass-args --arg-scope-and-set 0.7

ARG --global DOCKER_REPO
ARG GO_VERSION
ARG DEBIAN_VERSION
FROM --platform linux/amd64 golang:$GO_VERSION-$DEBIAN_VERSION
RUN set -ex && apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    pkg-config \
    cmake \
    curl \
    git \
    openjdk-11-jre \
    unzip \
    libtool \
    autoconf \
    zlib1g-dev \
    libssl-dev \
    clang \
    python3-pip
WORKDIR /tmp
ARG GRPC_VERSION
GIT CLONE --branch v$GRPC_VERSION.x git@github.com:grpc/grpc grpc

alpine-build:
    FROM --platform linux/amd64 alpine
    RUN apk add curl

grpc:
    WORKDIR /tmp/grpc
    LET bazel=/tmp/grpc/tools/bazel
    RUN $bazel build //external:protocol_compiler && \
        $bazel build //src/compiler:all && \
        $bazel build //test/cpp/util:grpc_cli
    # Install protoc required by envoyproxy/protoc-gen-validate package
    SAVE ARTIFACT /tmp/grpc/bazel-grpc/external/com_google_protobuf/src/google/protobuf /protobuf
    # Copy well known proto files required by envoyproxy/protoc-gen-validate package
    SAVE ARTIFACT /tmp/grpc/bazel-grpc/external/com_google_protobuf /com_google_protobuf
    SAVE ARTIFACT /tmp/grpc/bazel-bin/src/compiler /compiler
    SAVE ARTIFACT /tmp/grpc/bazel-bin/test/cpp/util /util

grpc-java:
    ARG GRPC_JAVA_VERSION
    GIT CLONE --branch v$GRPC_JAVA_VERSION.x git@github.com:grpc/grpc-java grpc-java
    WORKDIR /tmp/grpc-java
    LET bazel=/tmp/grpc/tools/bazel
    RUN $bazel build //compiler:grpc_java_plugin
    SAVE ARTIFACT /tmp/grpc-java/bazel-bin/compiler /compiler

protobuf-js:
    ARG PROTOBUF_JS_VERSION
    GIT CLONE --branch $PROTOBUF_JS_VERSION git@github.com:protocolbuffers/protobuf-javascript protobuf-javascript
    WORKDIR /tmp/protobuf-javascript
    LET bazel=/tmp/grpc/tools/bazel
    RUN $bazel build //generator:protoc-gen-js
    SAVE ARTIFACT /tmp/protobuf-javascript/bazel-bin/generator/protoc-gen-js /protoc-gen-js

googleapi:
    GIT CLONE https://github.com/googleapis/googleapis googleapis
    SAVE ARTIFACT googleapis/google /google

api-common-protos:
    GIT CLONE https://github.com/googleapis/api-common-protos api-common-protos
    SAVE ARTIFACT api-common-protos/google /google

uber:
    FROM +alpine-build
    ARG UBER_PROTOTOOL_VERSION
    RUN curl -fsSL "https://github.com/uber/prototool/releases/download/v${UBER_PROTOTOOL_VERSION}/prototool-$(uname -s)-$(uname -m)" \
    -o /usr/local/bin/prototool && \
    chmod +x /usr/local/bin/prototool
    SAVE ARTIFACT /usr/local/bin/prototool /prototool

go-binaries:
    # go install go-related bins
    ARG GO_PROTOC_GEN_GO_GRPC_VERSION
    RUN go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@$GO_PROTOC_GEN_GO_GRPC_VERSION

    # install protoc-gen-grpc-gateway and protoc-gen-openapiv2
    ARG GRPC_GATEWAY_VERSION
    RUN go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@${GRPC_GATEWAY_VERSION}
    RUN go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2@${GRPC_GATEWAY_VERSION}

    RUN go install github.com/gogo/protobuf/protoc-gen-gogo@latest
    RUN go install github.com/gogo/protobuf/protoc-gen-gogofast@latest

    RUN go install github.com/ckaznocha/protoc-gen-lint@latest

    RUN go install github.com/pseudomuto/protoc-gen-doc/cmd/protoc-gen-doc@latest

    RUN go install github.com/micro/micro/v3/cmd/protoc-gen-micro@latest

    ARG GO_ENVOYPROXY_PGV_VERSION
    RUN go install github.com/envoyproxy/protoc-gen-validate@v${GO_ENVOYPROXY_PGV_VERSION}

    # Add Ruby Sorbet types support (rbi)
    RUN go install github.com/coinbase/protoc-gen-rbi@latest

    RUN go install github.com/gomatic/renderizer/v2/cmd/renderizer@latest

    # Origin protoc-gen-go should be installed last, for not been overwritten by any other binaries(see #210)
    ARG GO_PROTOC_GEN_GO_VERSION
    RUN go install google.golang.org/protobuf/cmd/protoc-gen-go@${GO_PROTOC_GEN_GO_VERSION}

    ARG GO_MWITKOW_GPV_VERSION
    RUN go install github.com/mwitkow/go-proto-validators/protoc-gen-govalidators@v${GO_MWITKOW_GPV_VERSION}

    SAVE ARTIFACT /go/bin /bin
    SAVE ARTIFACT /go/pkg/mod/github.com/grpc-ecosystem/grpc-gateway/v2@${GRPC_GATEWAY_VERSION}/protoc-gen-openapiv2/options /options
    SAVE ARTIFACT /go/pkg/mod/github.com/envoyproxy/protoc-gen-validate@v${GO_ENVOYPROXY_PGV_VERSION} /protoc-gen-validate
    SAVE ARTIFACT /go/pkg/mod/github.com/mwitkow/go-proto-validators@v${GO_MWITKOW_GPV_VERSION} /go-proto-validators

scala:
    ARG SCALA_PB_VERSION
    RUN curl -fLO "https://github.com/scalapb/ScalaPB/releases/download/v${SCALA_PB_VERSION}/protoc-gen-scala-${SCALA_PB_VERSION}-linux-x86_64.zip" \
        && unzip protoc-gen-scala-${SCALA_PB_VERSION}-linux-x86_64.zip \
        && chmod +x /tmp/protoc-gen-scala
    SAVE ARTIFACT /tmp/protoc-gen-scala /protoc-gen-scala

grpc-web:
    ARG GRPC_WEB_VERSION
    RUN curl -fsSL "https://github.com/grpc/grpc-web/releases/download/${GRPC_WEB_VERSION}/protoc-gen-grpc-web-${GRPC_WEB_VERSION}-linux-x86_64" \
        -o /tmp/grpc_web_plugin && \
        chmod +x /tmp/grpc_web_plugin
    SAVE ARTIFACT /tmp/grpc_web_plugin /grpc_web_plugin

mypy:
    ARG MYPY_VERSION
    RUN pip3 install -t /opt/mypy-protobuf mypy-protobuf==${MYPY_VERSION}
    SAVE ARTIFACT /opt/mypy-protobuf /mypy-protobuf
    SAVE ARTIFACT /opt/mypy-protobuf/bin /bin

base-image:
    ARG DEBIAN_VERSION
    FROM --platform linux/amd64 debian:$DEBIAN_VERSION-slim
    RUN mkdir -p /usr/share/man/man1
    RUN set -ex && apt-get update && apt-get install -y --no-install-recommends \
        bash \
        curl \
        software-properties-common \
        ca-certificates \
        zlib1g \
        libssl1.1 \
        openjdk-11-jre \
        dos2unix \
        gawk \
        gnupg

    RUN mkdir -p /etc/apt/keyrings
    RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
    ARG NODE_VERSION
    RUN echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_VERSION.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
    RUN apt-get update && apt-get install nodejs -y

    # Add Node TypeScript support
    ARG NODE_GRPC_TOOLS_NODE_PROTOC_TS_VERSION
    ARG NODE_GRPC_TOOLS_VERSION
    ARG NODE_PROTOC_GEN_GRPC_WEB_VERSION
    RUN npm i -g grpc_tools_node_protoc_ts@$NODE_GRPC_TOOLS_NODE_PROTOC_TS_VERSION grpc-tools@$NODE_GRPC_TOOLS_VERSION protoc-gen-grpc-web@$NODE_PROTOC_GEN_GRPC_WEB_VERSION

    # Add TypeScript support
    ARG TS_PROTO_VERSION
    RUN npm i -g ts-proto@$TS_PROTO_VERSION

    COPY +googleapi/google /opt/include/google/.
    COPY +api-common-protos/google /opt/include/google/.
    COPY +grpc/com_google_protobuf /usr/local/bin/.
    COPY +grpc/protobuf /opt/include/google/protobuf/.
    COPY +grpc/compiler /usr/local/bin/.
    COPY +grpc/util /usr/local/bin/.
    COPY +grpc-java/compiler /usr/local/bin/.
    COPY +protobuf-js/protoc-gen-js /usr/local/bin/.
    COPY +uber/prototool /usr/local/bin/prototool
    COPY +go-binaries/bin /usr/local/bin/.
    COPY +grpc-web/grpc_web_plugin /usr/local/bin/grpc_web_plugin
    COPY +scala/protoc-gen-scala /usr/local/bin/.
    COPY +go-binaries/options /opt/include/protoc-gen-openapiv2/options/
    COPY +go-binaries/protoc-gen-validate /opt/include/
    COPY +go-binaries/go-proto-validators /opt/include/github.com/mwitkow/go-proto-validators/
    COPY --dir +mypy/mypy-protobuf /opt/mypy-protobuf
    COPY --dir +mypy/bin /usr/local/bin/
    ENV PYTHONPATH="${PYTHONPATH:+$PYTHONPATH:}/opt/mypy-protobuf/"
    COPY all/entrypoint.sh /usr/local/bin/.

    WORKDIR /defs
    ENTRYPOINT [ "entrypoint.sh" ]

protoc-all:
    FROM +base-image
    DO --pass-args +SAVE_IMAGE --IMAGE_NAME=protoc-all

protoc:
    FROM +base-image
    ENTRYPOINT [ "protoc", "-I/opt/include" ]
    DO --pass-args +SAVE_IMAGE --IMAGE_NAME=protoc

prototool:
    FROM +base-image
    ENTRYPOINT [ "prototool" ]
    DO --pass-args +SAVE_IMAGE --IMAGE_NAME=prototool

grpc-cli:
    FROM +base-image
    COPY ./cli/entrypoint.sh /entrypoint.sh
    RUN chmod +x /entrypoint.sh
    WORKDIR /run
    ENTRYPOINT [ "/entrypoint.sh" ]
    DO --pass-args +SAVE_IMAGE --IMAGE_NAME=grpc-cli

gen-grpc-gateway:
    FROM +base-image
    COPY gwy/templates /templates
    COPY gwy/generate_gateway.sh /usr/local/bin
    RUN chmod +x /usr/local/bin/generate_gateway.sh

    WORKDIR /defs
    ENTRYPOINT [ "generate_gateway.sh" ]
    DO --pass-args +SAVE_IMAGE --IMAGE_NAME=gen-grpc-gateway

SAVE_IMAGE:
    COMMAND
    ARG VERSION
    ARG GRPC_VERSION
    LET version=$VERSION
    IF [ -z $version ]
        ARG BUILD_VERSION
        RUN echo $GRPC_VERSION
        SET version="${GRPC_VERSION}_${BUILD_VERSION}"
    END
    ARG --required IMAGE_NAME
    SAVE IMAGE --push $DOCKER_REPO/$IMAGE_NAME:$version
    ARG LATEST
    IF [ "$LATEST" = "true" ]
        SAVE IMAGE --push $DOCKER_REPO/$IMAGE_NAME:latest
    END

all:
    BUILD +protoc-all
    BUILD +protoc
    BUILD +grpc-cli
    BUILD +prototool
    BUILD +gen-grpc-gateway

test:
    BUILD +test-protoc-all
    BUILD +test-gwy

test-protoc-all:
    FROM earthly/dind:alpine-3.18
    RUN apk add bash go
    WORKDIR /tests/all
    COPY --dir ./all/test .
    COPY ./all/test.sh .
    WORKDIR /tests
    WITH DOCKER --load test=+protoc-all
        RUN CONTAINER=test bash ./all/test.sh
    END

test-gwy:
    FROM earthly/dind:alpine-3.18
    RUN apk add bash go
    WORKDIR /tests/gwy
    COPY --dir ./gwy/test .
    COPY ./gwy/test.sh .
    WORKDIR /tests
    WITH DOCKER --load test=+gen-grpc-gateway
        RUN CONTAINER=test bash ./gwy/test.sh
    END