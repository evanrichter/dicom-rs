# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    bash -c "pushd ul/fuzz && cargo +nightly -Z sparse-registry fuzz build && popd" && \
    bash -c "pushd object/fuzz && cargo +nightly -Z sparse-registry fuzz build && popd" && \
    mv ul/fuzz/target/x86_64-unknown-linux-gnu/release/pdu_roundtrip /pdu_roundtrip && \
    mv object/fuzz/target/x86_64-unknown-linux-gnu/release/open_file /open_file && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd ul/fuzz && cargo +nightly -Z sparse-registry build --release && popd" && \
    bash -c "pushd object/fuzz && cargo +nightly -Z sparse-registry build --release && popd" && \
    mv ul/fuzz/target/release/pdu_roundtrip /pdu_roundtrip_no_inst && \
    mv object/fuzz/target/release/open_file /open_file_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /pdu_roundtrip /pdu_roundtrip_no_inst /
COPY --from=builder /open_file /open_file_no_inst /

ENV ASAN_OPTIONS=allocator_may_return_null=1:max_allocation_size_mb=40
