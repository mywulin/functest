FROM opnfv/functest-core

ARG BRANCH=master
ARG OPENSTACK_TAG=stable/pike
ARG ODL_TAG=85448c9d97b89989488e675b29b38ac42d8674e4
ARG RALLY_TAG=stable/0.10
ARG OS_FAULTS_TAG=0.1.16
ARG REFSTACK_TARGET=2017.09
ARG PATROLE_TAG=0.2.0

COPY thirdparty-requirements.txt thirdparty-requirements.txt
RUN apk --no-cache add --virtual .build-deps --update \
        python-dev build-base linux-headers libffi-dev \
        openssl-dev libjpeg-turbo-dev && \
    wget -q -O- https://git.openstack.org/cgit/openstack/requirements/plain/upper-constraints.txt?h=$OPENSTACK_TAG | \
        sed -E s/^tempest==+\(.*\)$/-e\ git+https:\\/\\/github.com\\/openstack\\/tempest@\\1#egg=tempest/ \
        > upper-constraints.txt && \
    pip install --no-cache-dir --src /src -cupper-constraints.txt \
        -chttps://git.opnfv.org/functest/plain/upper-constraints.txt?h=$BRANCH \
        -e git+https://github.com/openstack/requirements@$OPENSTACK_TAG#egg=openstack_requirements && \
    git clone --depth 1 https://github.com/openstack/os-faults.git -b $OS_FAULTS_TAG /src/os-faults && \
    update-requirements -s --source /src/openstack-requirements /src/os-faults/ && \
    git clone --depth 1 https://github.com/openstack/rally.git -b $RALLY_TAG /src/rally && \
    update-requirements -s --source /src/openstack-requirements /src/rally/ && \
    git clone https://github.com/openstack/patrole.git /src/patrole && \
    (cd /src/patrole && git checkout $PATROLE_TAG) && \
    update-requirements -s --source /src/openstack-requirements /src/patrole/ && \
    pip install --no-cache-dir --src /src -cupper-constraints.txt \
        -chttps://git.opnfv.org/functest/plain/upper-constraints.txt?h=$BRANCH \
        tempest /src/os-faults && \
    pip install --no-cache-dir --src /src -cupper-constraints.txt \
        -chttps://git.opnfv.org/functest/plain/upper-constraints.txt?h=$BRANCH \
        /src/rally /src/patrole -rthirdparty-requirements.txt && \
    git clone https://git.opendaylight.org/gerrit/p/integration/test.git /src/odl_test && \
    (cd /src/odl_test && git checkout $ODL_TAG) && \
    virtualenv --system-site-packages /src/tempest/.venv && \
    pip install --no-cache-dir --src /src -cupper-constraints.txt \
        -chttps://git.opnfv.org/functest/plain/upper-constraints.txt?h=$BRANCH \
        -e git+https://github.com/openstack/neutron.git@$OPENSTACK_TAG#egg=neutron && \
    pip install --no-cache-dir --src /src -cupper-constraints.txt \
        -chttps://git.opnfv.org/functest/plain/upper-constraints.txt?h=$BRANCH \
        -r /src/neutron/test-requirements.txt && \
    rm -r upper-constraints.txt thirdparty-requirements.txt /src/odl_test/.git \
        /src/os-faults /src/rally && \
    mkdir -p /etc/rally && \
    printf "[database]\nconnection = 'sqlite:////var/lib/rally/database/rally.sqlite'" > /etc/rally/rally.conf && \
    mkdir -p /var/lib/rally/database && rally db create && \
    mkdir -p /home/opnfv/functest/data/refstack && \
    wget "https://refstack.openstack.org/api/v1/guidelines/${REFSTACK_TARGET}/tests?target=compute&type=required&alias=true&flag=false" \
        -O /home/opnfv/functest/data/refstack/defcore.txt && \
    mkdir -p /etc/neutron /etc/glance && \
    wget -q -O /etc/neutron/policy.json https://raw.githubusercontent.com/openstack/neutron/$OPENSTACK_TAG/etc/policy.json && \
    wget -q -O /etc/glance/policy.json https://raw.githubusercontent.com/openstack/glance/$OPENSTACK_TAG/etc/policy.json && \
    apk del .build-deps
COPY testcases.yaml /usr/lib/python2.7/site-packages/xtesting/ci/testcases.yaml
CMD ["run_tests", "-t", "all"]
