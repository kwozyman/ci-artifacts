FROM registry.access.redhat.com/ubi8/ubi

MAINTAINER Red Hat Edge Ecosystem Team <assisted-installer@redhat.com>

LABEL 	io.k8s.display-name="Red Hat Edge Ecosystem CI artifacts" \
      	io.k8s.description="An image for running Ansible artifacts for RedHat Edge Ecosystem CI" \
 	name="ci-artifacts" \
	url="https://github.com/rh-ecosystem-edge/ci-artifacts"

COPY requirements.txt /

# Install openshift-ansible RPMs and some debugging tools
RUN yum install -y \
		glibc-langpack-en \
		go git make jq vim wget rsync time\
		python3 python3-devel python3-pip python3-setuptools && \
	python3 -m pip install --no-cache-dir --upgrade setuptools pip wheel && \
	python3 -m pip install --no-cache-dir -r /requirements.txt && \
	yum clean all && \
	rm -rf $HOME/.cache && \
	rm -rf /var/cache/yum


# Install dependencies: `oc`
ARG OCP_CLI_VERSION=latest
ARG OCP_CLI_URL=https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${OCP_CLI_VERSION}/openshift-client-linux.tar.gz
RUN curl ${OCP_CLI_URL} | tar xfz - -C /usr/local/bin oc

# Install dependencies: `ocm`
ARG OCM_CLI_VERSION=v0.1.60
ARG OCM_CLI_URL=https://github.com/openshift-online/ocm-cli/releases/download/${OCM_CLI_VERSION}/ocm-linux-amd64
RUN curl -L ${OCM_CLI_URL} --output /usr/local/bin/ocm
RUN chmod +x /usr/local/bin/ocm

# Install dependencies: `helm`
ARG HELM_VERSION=v3.5.1
ARG HELM_URL=https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz
RUN curl ${HELM_URL} | tar xfz - -C /usr/local/bin --strip-components 1 linux-amd64/helm


# Install dependencies: `operator-sdk`
ARG OPERATOR_SDK_VERSION=v1.6.2
ARG OPERATOR_SDK_URL=https://github.com/operator-framework/operator-sdk/releases/download/${OPERATOR_SDK_VERSION}
RUN cd /usr/local/bin \
 && curl -LO ${OPERATOR_SDK_URL}/operator-sdk_linux_amd64 \
 && mv operator-sdk_linux_amd64 operator-sdk \
 && chmod +x operator-sdk

# Set up Ansible
RUN mkdir -p /etc/ansible \
    && echo "localhost ansible_connection=local" > /etc/ansible/hosts
COPY config/ansible.cfg /etc/ansible/ansible.cfg
COPY config/inventory/hosts /etc/ansible/inventory/hosts

RUN sed -i 's|roles/|/opt/ci-artifacts/roles/|' /etc/ansible/ansible.cfg \
 && sed -i 's|callback_plugins = ../|callback_plugins = /opt/ci-artifacts/src/|' /etc/ansible/ansible.cfg

# Set up the runner user
ENV USER_NAME=edge-ci-runner \
    USER=edge-ci-runner \
    HOME=/opt/ci-artifacts/src/ \
    ANSIBLE_CONFIG="/etc/ansible/ansible.cfg" \
    INSIDE_CI_IMAGE="y"

COPY . ${HOME}/

# Prepare the CI `run` entrypoint
RUN echo -e '#!/usr/bin/env bash \n\
exec ${HOME}/testing/run "$@" \n\
' > /usr/local/bin/run; chmod ugo+x /usr/local/bin/run

# Prepare osde2e results folder
RUN mkdir /test-run-results && chmod 777 /test-run-results

# Ensure directory permissions are properly set
RUN mkdir -p ${HOME}/.ansible/tmp && chmod 777 ${HOME} -R

WORKDIR ${HOME}

ENTRYPOINT ["bash"]
CMD ["run"]
