# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# FROM eclipse-temurin:19@sha256:17e3d3b61ca4a7606490f596feb77f69980939fecacf91a1f13ea7b17147058f as builder

FROM --platform=linux/amd64 eclipse-temurin:19.0.1_10-jdk-jammy as builder

WORKDIR /app

COPY ["build.gradle", "gradlew", "./"]
COPY gradle gradle
RUN chmod +x gradlew
RUN ./gradlew downloadRepos

COPY . .
RUN chmod +x gradlew
RUN ./gradlew installDist

# FROM eclipse-temurin:19.0.1_10-jre-alpine@sha256:1aa167ab4f1498130e04bed5d6a83fed23c1fd8e3df8589723bc876770dd6a3a as without-grpc-health-probe-bin

FROM eclipse-temurin:19.0.1_10-jre-jammy as without-grpc-health-probe-bin

RUN apt-get -y update && apt-get install -qqy \
    wget \
    && rm -rf /var/lib/apt/lists/*

# RUN apk add --no-cache ca-certificates

# Download Stackdriver Profiler Java agent
# RUN mkdir -p /opt/cprof && \
#     wget -q -O- https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent_alpine.tar.gz \
#     | tar xzv -C /opt/cprof && \
#     rm -rf profiler_java_agent.tar.gz

WORKDIR /app
COPY --from=builder /app .

EXPOSE 9555
ENTRYPOINT ["/app/build/install/hipstershop/bin/AdService"]

FROM without-grpc-health-probe-bin

ARG TARGETOS 
ARG TARGETARCH
RUN echo ${TARGETOS}
RUN echo ${TARGETARCH}

# renovate: datasource=github-releases depName=grpc-ecosystem/grpc-health-probe
RUN GRPC_HEALTH_PROBE_VERSION=v0.4.14 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-${TARGETARCH} && \
    chmod +x /bin/grpc_health_probe
