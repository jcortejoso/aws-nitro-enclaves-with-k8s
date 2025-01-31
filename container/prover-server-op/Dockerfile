# Copyright 2022 Amazon.com, Inc. or its affiliates. All Rights Reserved.
# SPDX-License-Identifier: Apache-2.0

######## full image ########

FROM amazonlinux:2 as full_image

RUN amazon-linux-extras install aws-nitro-enclaves-cli && \
    yum install aws-nitro-enclaves-cli-devel jq -y

WORKDIR /ne-deps

# Copy only the required binaries to /ne-deps folder.
#
RUN BINS="\
    /usr/bin/nitro-cli \
    /usr/bin/nitro-enclaves-allocator \
    /usr/bin/jq \
    " && \
    for bin in $BINS; do \
        { echo "$bin"; ldd "$bin" | grep -Eo "/.*lib.*/[^ ]+"; } | \
            while read path; do \
                mkdir -p ".$(dirname $path)"; \
                cp -fL "$path" ".$path"; \
            done \
    done

# Prepare other required files and folders for the final image.
#
RUN \
    mkdir -p /ne-deps/etc/nitro_enclaves && \
    mkdir -p /ne-deps/run/nitro_enclaves && \
    mkdir -p /ne-deps/var/log/nitro_enclaves && \
    cp -rf /usr/share/nitro_enclaves/ /ne-deps/usr/share/ && \
    cp -f /etc/nitro_enclaves/allocator.yaml /ne-deps/etc/nitro_enclaves/allocator.yaml

######## prover-server-op image ########

FROM amazonlinux:2 as image

# Copying dependencies of the enclave apps from the 'full_image'
# to shrink the final image size.
#
COPY --from=full_image / /
# COPY --from=full_image /ne-deps /
# COPY --from=full_image /ne-deps/etc/nitro_enclaves /etc/
# COPY --from=full_image /ne-deps/run/nitro_enclaves /run/
# COPY --from=full_image /ne-deps/var/log/nitro_enclaves /var/log/
# COPY --from=full_image /ne-deps/usr/share /usr/
# COPY --from=full_image /ne-deps/usr/bin /usr/
# COPY --from=full_image /ne-deps/usr/lib64 /usr/

COPY prover-server-op/allocator.yaml /etc/nitro_enclaves/allocator.yaml
COPY prover-server-op/allocator.yaml /ne-deps/etc/nitro_enclaves/allocator.yaml
COPY prover-server-op/socat /usr/bin/socat

COPY prover-server-op/prover-server-op-0.2.eif /home
COPY prover-server-op/proxy-bin /home/proxy-bin
COPY prover-server-op/run.sh  /home

CMD ["/home/run.sh"]
