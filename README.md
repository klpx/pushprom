
# Pushprom

Pushprom is a proxy (HTTP/UDP) to the [Prometheus](https://prometheus.io/) Go client.

Prometheus doesn't offer a PHP client and PHP clients are hard to implement because they would need to keep track of state and PHP setups generally don't encourage that. That's why we built Pushprom.

## Installing

Execute the following command:

```bash
make release
```

Or, alternatively, to build a Docker container:

```bash
make container
```

## Usage

Running Pushprom is as easy as executing `pushprom` on the command line.

```
$ pushprom
2016/01/15 16:35:25 main.go:26: exposing metrics on http://127.0.0.1::9092/metrics
2016/01/15 16:35:25 http.go:34: listening for stats on http://127.0.0.1:9091
2016/01/15 16:35:25 udp.go:10: listening for stats UDP on port :9090
```

Use the `-h` flag to get help information.

```
$ pushprom -h
Usage of bin/pushprom:
  -debug
        Log debugging messages.
  -http-listen-address string
        The address to listen on for http stat and telemetry requests. (default ":9091")
  -udp-listen-address string
        The address to listen on for udp stats requests. (default ":9090")
```

Pushprom accepts HTTP and UDP requests. The payloads are in JSON. Here is a full example:

```json
{
      "type": "gauge",
      "name": "trees",
      "help": "the amount of trees in the forest.",
      "method": "add",
      "value": 3002,
      "labels": {
            "species": "araucaria angustifolia",
            "job": "tree-counter-bot"
      }
}
```

When Pushprom receives this payload (from now on called Delta) it tries to register the metric with type **Gauge** named **trees** and then apply the operation **add** with value **3002** on it.

## Protocol support

You can use HTTP requests and UDP packages to push deltas to Pushprom.

### HTTP

When using HTTP you should do a `POST /`.

Example:

```bash
curl -H "Content-type: application/json" -X POST -d '{"type": "counter", "name": "gophers", "help": "little burrowing rodents", "method": "inc"}' http://127.0.0.1:9091/
```

### UDP

*You move fast and break things.*

Example:

```bash
echo "{\"type\": \"counter\", \"name\": \"gophers\", \"help\": \"little burrowing rodents\", \"method\": \"inc\"}" | nc -u -w1 127.0.0.1 9090
```

## Caveats

In the Prometheus Go client you can not register a metric with the same **name** and different **help** or **labels**. For example: you register a metric with name `gophers` and with help `little rodents` and a little later you think "but they are also burrowing animals!". When you change the help string and push the same metric it won't work: you need to reboot Pushprom.

## Clients

We currently offer two flavors of PHP clients for Pushprom:
* [PHP](https://github.com/messagebird/pushprom-php-client)
* [Yii 2](https://github.com/messagebird/pushprom-yii2-client).

## Alternatives

### Pushgateway

[Pushgateway](https://github.com/prometheus/pushgateway) is a metrics cache for Prometheus. It's explicitly not an aggregator, which is the most distinct difference with Pushprom.

# Tests

```
go tests
```

## License

Pushprom is licensed under [The BSD 2-Clause License](http://opensource.org/licenses/BSD-2-Clause). Copyright (c) 2016, MessageBird



