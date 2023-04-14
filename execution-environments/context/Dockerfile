ARG EE_BASE_IMAGE=quay.io/ansible/awx-ee:latest
ARG EE_BUILDER_IMAGE=quay.io/ansible/ansible-builder:latest

FROM $EE_BASE_IMAGE as galaxy
ARG ANSIBLE_GALAXY_CLI_COLLECTION_OPTS=--pre
ARG ANSIBLE_GALAXY_CLI_ROLE_OPTS=
USER root

ADD _build /build
WORKDIR /build


FROM $EE_BUILDER_IMAGE as builder
ADD _build/requirements.txt requirements.txt
RUN ansible-builder introspect --sanitize --user-pip=requirements.txt --write-bindep=/tmp/src/bindep.txt --write-pip=/tmp/src/requirements.txt
RUN assemble

FROM $EE_BASE_IMAGE
USER root
RUN pip3 install --upgrade pip setuptools
COPY --from=builder /output/ /output/
RUN /output/install-from-bindep && rm -rf /output/wheels
RUN ls -la /etc
LABEL ansible-execution-environment=true
