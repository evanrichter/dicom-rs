# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    cargo +nightly -Z sparse-registry fuzz build --fuzz-dir ul/fuzz && \
    cargo +nightly -Z sparse-registry fuzz build --fuzz-dir object/fuzz && \
    mv ul/fuzz/target/x86_64-unknown-linux-gnu/release/pdu_roundtrip /pdu_roundtrip && \
    mv object/fuzz/target/x86_64-unknown-linux-gnu/release/open_file /open_file && \
    echo done

RUN echo building non-instrumented harnesses && \
    cargo +nightly -Z sparse-registry fuzz build --fuzz-dir ul/fuzz --disable-instrumentation && \
    cargo +nightly -Z sparse-registry fuzz build --fuzz-dir object/fuzz --disable-instrumentation && \
    mv ul/fuzz/target/x86_64-unknown-linux-gnu/release/pdu_roundtrip /pdu_roundtrip_no_inst && \
    mv object/fuzz/target/x86_64-unknown-linux-gnu/release/open_file /open_file_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /pdu_roundtrip  /pdu_roundtrip_no_inst /
COPY --from=builder /open_file  /open_file_no_inst /
