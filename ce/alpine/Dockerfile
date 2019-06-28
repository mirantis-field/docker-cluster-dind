# ======================================
# Build Stage
# ======================================
FROM library/docker:19.03.0-rc3-dind

ARG GITHUB_TOKEN

ENV FETCH_VERSION v0.3.5
ENV DOCKER_CLUSTER_ASSET_FILE_NAME cluster-linux-amd64.zip
ENV DOCKER_CLUSTER_VERSION v1.0.0

WORKDIR /tmp

ADD https://github.com/gruntwork-io/fetch/releases/download/${FETCH_VERSION}/fetch_linux_amd64 /usr/bin/fetch

RUN chmod u+x /usr/bin/fetch \
&& fetch --github-oauth-token=${GITHUB_TOKEN} --repo="https://github.com/docker/cluster" --tag=${DOCKER_CLUSTER_VERSION} --release-asset=${DOCKER_CLUSTER_ASSET_FILE_NAME} /tmp \
&& unzip /tmp/${DOCKER_CLUSTER_ASSET_FILE_NAME} 





# ======================================
# Runtime Stage
# ======================================
FROM library/docker:19.03.0-rc3-dind

COPY --from=0 /tmp/docker-cluster /home/cluster/.docker/cli-plugins/

RUN apk add --no-cache libc6-compat \
&& addgroup --gid 1000 cluster \
&& adduser -D -G cluster -u 1000 cluster \
&& adduser cluster root \
&& mkdir -p /home/cluster/.docker/cli-plugins \
&& chown -R cluster:cluster /home/cluster

WORKDIR /home/cluster

