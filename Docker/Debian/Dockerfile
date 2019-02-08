### At first perform source build ###
FROM starwarsfan/cmake-builder-debian:1.0 as build
MAINTAINER Yves Schumann <yves@eisfair.org>

COPY . /demo-app

RUN mkdir /demo-app/build \
 && cd /demon-app/build \
 && cmake ..

### Now upload binaries to GitHub ###
FROM starwarsfan/github-release-uploader:1.0
MAINTAINER Yves Schumann <yves@eisfair.org>

ARG GITHUB_TOKEN=1234567
ARG RELEASE=latest
ARG REPOSITORY=cmake-buildtest
ARG GIT_COMMIT=unknown
ARG REPLACE_EXISTING_ARCHIVE=''
ENV ARCHIVE=cmake-buildtest-${RELEASE}-${GIT_COMMIT}-Debian.tgz

RUN mkdir -p /filesToUpload/usr/local/bin

# Maybe copy more from the builder...
COPY --from=build /demo-app/build/hello-world /filesToUpload/usr/local/bin/hello-world

RUN cd /filesToUpload \
 && tar czf ${ARCHIVE} . \
 && github-release upload \
        --user starwarsfan \
        --security-token "${GITHUB_TOKEN}" \
        --repo "${REPOSITORY}" \
        --tag "${RELEASE}" \
        --name "${ARCHIVE}" \
        --file "/filesToUpload/${ARCHIVE}" \
        ${REPLACE_EXISTING_ARCHIVE} \
 && export GITHUB_TOKEN=---
