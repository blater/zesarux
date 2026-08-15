# ZEsarUX headless automation fork

`main` mirrors upstream ZEsarUX. `integration/zrcp-automation` combines the
patches described below. Each patch also has its own feature branch.

This fork adds a small set of remote-control improvements for safer, more
deterministic automated testing:

- ZRCP listens on loopback by default, keeping the unauthenticated control
  interface off external network interfaces while still allowing an explicit
  bind address.
- CPU stepping works reliably with the null video driver, so breakpoint-driven
  tests do not need a graphical display.
- Partial t-state counter resets return an acknowledgement, allowing test tools
  to synchronize timing measurements without guessing when a command finished.

For the emulator's complete documentation, supported machines, build guidance,
and licence information, see the
[original ZEsarUX project and README created by César Hernández Bañó](https://github.com/chernandezba/zesarux/blob/main/README.md).

## Headless use

Build the emulator normally from its `src` directory:

```sh
cd /path/to/zesarux/src
./configure
make -j4
```

Start a ZX Spectrum Next instance without video or audio and expose ZRCP only
on the loopback interface:

```sh
./zesarux --noconfigfile --machine tbblue --vo null --ao null \
  --nosplash --nowelcomemessage --quickexit \
  --enable-remoteprotocol --remoteprotocol-host 127.0.0.1 \
  --remoteprotocol-port 10000
```

An automation process then opens a TCP connection to `127.0.0.1:10000`, waits
for the ZRCP prompt, and sends newline-terminated commands. A typical controlled
run uses commands such as:

```text
snapshot-load /absolute/path/to/program.nex
enter-cpu-step
get-registers
reset-tstates-partial-ack
exit-cpu-step
exit-emulator
```

`enter-cpu-step` provides a deterministic pause before registers, memory, or
breakpoints are inspected. `reset-tstates-partial-ack` resets the measurement
counter and confirms completion in its response, allowing execution timing to
start from a known point. The controller should wait for each response prompt
before sending the next command and should always terminate or reap the
emulator process on timeout.

ZRCP is unauthenticated. Keep it bound to `127.0.0.1` unless the surrounding
system provides an appropriate trusted network boundary.
