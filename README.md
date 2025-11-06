# arrow-generator-stream-repro

This repo is the reproducer for the regression in pyarrow 22.0.0 `GeneratorStream` probably related
to https://github.com/apache/arrow/commit/25f8f008061b978137a2c1e5bc934d07fae56e3e

The code is the example server and client from https://github.com/apache/arrow/tree/main/python/examples/flight with
slight changes to the server denoted by
`# START DIFF` and `# END DIFF` comments:

* definition of a simple iterator that returns table data (in real applications this iterator is a bit more involved of
  course)
* do_get now returns a GeneratorStream with this iterator

## Repro steps

1. run `uv sync`
2. run `uv run server.py`
3. in another terminal run `uv run client.py put localhost:5005 cities.csv`
4. run `uv run client.py get -p cities.csv localhost:5005`

The code works on pyarrow 21 correctly, however on 22 it breaks with the following error:

```
Ticket: <pyarrow.flight.Ticket ticket=b"(1, None, (b'cities.csv',))">
<pyarrow.flight.Location b'grpc+tcp://localhost:5005'>
Traceback (most recent call last):
  File "/Users/hello/arrow-generator-stream-repro/client.py", line 192, in <module>
    main()
  File "/Users/hello/arrow-generator-stream-repro/client.py", line 188, in main
    commands[args.action](args, client, connection_args)
  File "/Users/hello/arrow-generator-stream-repro/client.py", line 103, in get_flight
    df = reader.read_pandas()
         ^^^^^^^^^^^^^^^^^^^^
  File "pyarrow/ipc.pxi", line 704, in pyarrow.lib._ReadPandasMixin.read_pandas
  File "pyarrow/_flight.pyx", line 1188, in pyarrow._flight.FlightStreamReader.read_all
  File "pyarrow/_flight.pyx", line 1191, in pyarrow._flight.FlightStreamReader.read_all
  File "pyarrow/_flight.pyx", line 58, in pyarrow._flight.check_flight_status
pyarrow._flight.FlightServerError: Unknown error: Writer should be initialized before reading Next batches. Detail: Python exception: Traceback (most recent call last):
  File "pyarrow/_flight.pyx", line 2129, in pyarrow._flight._data_stream_next
  File "pyarrow/_flight.pyx", line 78, in pyarrow._flight.check_flight_status
  File "pyarrow/error.pxi", line 92, in pyarrow.lib.check_status
pyarrow.lib.ArrowException: Unknown error: Writer should be initialized before reading Next batches
. gRPC client debug context: UNKNOWN:Error received from peer ipv6:%5B::1%5D:5005 {grpc_message:"Unknown error: Writer should be initialized before reading Next batches. Detail: Python exception: Traceback (most recent call last):\n  File \"pyarrow/_flight.pyx\", line 2129, in pyarrow._flight._data_stream_next\n  File \"pyarrow/_flight.pyx\", line 78, in pyarrow._flight.check_flight_status\n  File \"pyarrow/error.pxi\", line 92, in pyarrow.lib.check_status\npyarrow.lib.ArrowException: Unknown error: Writer should be initialized before reading Next batches\n", grpc_status:2, created_time:"2025-11-06T16:37:18.182185+01:00"}. Client context: OK
```
