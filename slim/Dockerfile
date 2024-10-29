# FROM --platform=linux/amd64 rust:1.82-slim-bookworm

# ENV NVM_DIR=/root/.nvm
# ENV NVM_VERSION=v0.40.1
# ENV NODE_VERSION=22.10.0

# RUN apt -yq update
# RUN apt -yqq install --no-install-recommends curl ca-certificates libunwind-dev git

# # Install Node.js using nvm
# ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin:${PATH}"
# RUN curl --fail -sSf https://raw.githubusercontent.com/creationix/nvm/${NVM_VERSION}/install.sh | bash
# RUN . "${NVM_DIR}/nvm.sh" && nvm install ${NODE_VERSION}
# RUN . "${NVM_DIR}/nvm.sh" && nvm use v${NODE_VERSION}
# RUN . "${NVM_DIR}/nvm.sh" && nvm alias default v${NODE_VERSION}

# # Install dfx
# ENV DFXVM_INIT_YES=true
# ENV DFX_VERSION=0.24.1
# RUN curl -fsSL https://internetcomputer.org/install.sh | sh
# ENV PATH="/root/.local/share/dfx/bin:$PATH"
# RUN dfx cache install

# # Add wasm32-unknown-unknown target
# RUN rustup target add wasm32-unknown-unknown

# # Install candid-extractor
# RUN curl -fsSL https://github.com/dfinity/candid-extractor/releases/download/0.1.5/candid-extractor-x86_64-unknown-linux-gnu.tar.gz -o candid-extractor.tar.gz
# RUN tar -xvf candid-extractor.tar.gz && rm candid-extractor.tar.gz
# RUN mv candid-extractor /usr/local/bin

# # Clean apt
# RUN apt-get autoremove && apt-get clean

FROM node:latest

RUN apt-get update \
    && apt-get install -y libssl-dev curl git jq nano

WORKDIR /app

# Create the blessed user and create the data directory
RUN adduser --disabled-password --gecos "" --uid 1001 blessed \
    && mkdir /data \
    && chown -R blessed:blessed /app \
    && chown -R blessed:blessed /data

USER blessed

ENV DATA_DIR=/data
ENV PATH=/home/blessed/bin:${PATH}

RUN mkdir -p /home/blessed/bin && mkdir -p /home/blessed/.mops/bin

ENV PATH=/home/blessed/.mops/bin:/home/blessed/bin:${PATH}

# Install didc
RUN wget https://github.com/dfinity/candid/releases/download/2024-05-14/didc-linux64 \
    && chmod +x didc-linux64 \
    && mv didc-linux64 /home/blessed/bin/didc

# Install ic-repl
RUN wget https://github.com/dfinity/ic-repl/releases/download/0.7.4/ic-repl-linux64 \
    && chmod +x ic-repl-linux64 \
    && mv ic-repl-linux64 /home/blessed/bin/ic-repl

# Install chunked-sync
RUN wget https://github.com/dfinity-ryancroote/ic-asset/releases/download/0.0.10/chunked-sync-linux64 \
    && chmod +x chunked-sync-linux64 \
    && mv chunked-sync-linux64 /home/blessed/bin/chunked-sync

# Install mops-cli
RUN wget https://github.com/chenyan2002/mops-cli/releases/download/0.1.0/mops-cli-linux64 \
    && chmod +x mops-cli-linux64 \
    && mv mops-cli-linux64 /home/blessed/bin/mops-cli

# Install dfx
ENV DFXVM_INIT_YES=true
ENV DFX_VERSION=0.24.0
RUN curl -fsSL https://internetcomputer.org/install.sh | sh
ENV PATH="/home/blessed/.local/share/dfx/bin:$PATH"
RUN dfx cache install

# Install cargo, add wasm32 target
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
ENV PATH="/home/blessed/.cargo/bin:$PATH"
RUN rustup target add wasm32-unknown-unknown

# Install candid-extractor
RUN cargo install candid-extractor

COPY --chown=blessed:blessed bin/deploy bin/deploy

# 0. Make a build directory.
# 1. Copy the mops.toml and dummy.mo to that build directory.
# 2. Run mops-cli build on dummy.mo as that installs the motoko compiler and base libraries in mops.toml.
# 3. Runs mops-cli update --moc to update to the latest version of the motoko compiler.
# 4. Remove the build directory.
RUN mkdir -p /tmp/build \
    && cp bin/deploy/mops.toml /tmp/build/mops.toml \
    && cp bin/deploy/dummy.mo /tmp/build/dummy.mo \
    && cd /tmp/build \
    && mops-cli build dummy.mo \
    && mops-cli update --moc \
    && rm -rf /tmp/build

# Note: Everything beyond this point will need to be re-built everytime
COPY --chown=blessed:blessed . .

ENV TZ=UTC

RUN npm install \
    && mkdir /data/sessions \
    && npm run prepare \
    && npm run build \
    && cd sandbox \
    && npm install
