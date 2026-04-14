# wb_streamer

Traces from the wb_streamer core. We used version `1.1`.

## Changes needed

The original `.core` file uses CAPI1 format, which FuseSoC 2.x no longer supports.
`wb_streamer.core` was rewritten from CAPI1 to CAPI2 following the instructions here:
https://fusesoc.readthedocs.io/en/stable/ref/migrations.html#migrating-from-capi1-to-capi2

The `fusesoc-cores` library also bundles its own copy of `wb_streamer` in CAPI1 format,
which shadows the local one. We deleted `fusesoc_libraries/fusesoc-cores/wb_streamer` to fix this.

## Generating waveforms

There are two testbenches, one for each direction. The process is the same for both.

```sh
uv run fusesoc library add fusesoc-cores https://github.com/fusesoc/fusesoc-cores

# writer (stream -> memory)
uv run fusesoc run --flag stream_bfm --tool=icarus --target=sim wb_streamer

# reader (memory -> stream)
uv run fusesoc run --flag stream_bfm --tool=icarus --target=sim_reader wb_streamer
```

We then add waveform dumping to the generated testbench right before `@(negedge rst)`:

```verilog
$dumpfile("wb_stream_writer_tb.fst"); $dumpvars(0);
```

The testbenches are at:
- `build/wb_streamer_1.1/sim-icarus/src/wb_streamer_1.1/bench/wb_stream_writer_tb.v`
- `build/wb_streamer_1.1/sim_reader-icarus/src/wb_streamer_1.1/bench/wb_stream_reader_tb.v`

Then rebuild (the generated Makefile doesn't track dependencies, so we have to delete the binary first):

```sh
cd build/wb_streamer_1.1/sim-icarus && rm wb_streamer_1.1 && make run
gtkwave wb_stream_writer_tb.fst
```
