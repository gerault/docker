### build environment
FROM golang:1.11.0-alpine3.8 AS build

ARG TEGOLA_VERSION="integrate_geom_makevalid"
#ARG TEGOLA_VERSION="master"

ARG TEGOLA_SOURCE="https://github.com/go-spatial/tegola.git"
ARG TEGOLA_BUILDFLAGS="-w -X github.com/go-spatial/tegola/cmd/tegola/cmd.Version=${TEGOLA_VERSION}"

### update apk and existing packages
RUN apk upgrade --update &&\
    ### build packages
    apk add \
        musl-dev=1.1.19-r10 \
        gcc=6.4.0-r9 \
        git

### tegola: source download
RUN mkdir -p /go/src/github.com/go-spatial &&\
    cd /go/src/github.com/go-spatial &&\
    git clone --branch $TEGOLA_VERSION --depth 1 $TEGOLA_SOURCE tegola
    
### additional dependency geom: source download
RUN cd /go/src/github.com/go-spatial &&\
    git clone --branch gdey_delaunay --depth 1 https://github.com/go-spatial/geom.git geom

### tegola: build + install
RUN cd /go/src/github.com/go-spatial/tegola/cmd/tegola &&\
    go build -v -ldflags "$TEGOLA_BUILDFLAGS" -gcflags "-N -l" -o /opt/tegola &&\
    chmod a+x /opt/tegola

### Create minimal deployment image, just alpine & the binary
FROM alpine:3.8

RUN apk upgrade --update &&\
    ### runtime packages
    apk add --virtual .run-deps \
        ca-certificates &&\
    ### clean up files
    rm -rf /var/cache/apk/*

### add tegola binary
COPY --from=build /opt/tegola /opt/

### set runtime environment
WORKDIR /opt
ENTRYPOINT ["/opt/tegola"]