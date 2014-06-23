GELF Client
===========

A Java GELF client library with support for different transports.

Available transports:

* TCP
* UDP

The library uses [Netty v4](http://netty.io/) to handle all network related
tasks and [Jackson](https://github.com/FasterXML/jackson) for JSON encoding.

## Usage

### Maven Dependency

```xml
<dependency>
  <groupId>org.graylog2</groupId>
  <artifactId>gelfclient</artifactId>
  <version>1.0.0-SNAPSHOT</version>
</dependency>
```

### Example

```java
public class Application {
    public static void main(String[] args) {
        boolean blocking = false;
        GelfConfiguration config = new GelfConfiguration();

        // Required settings
        config.setTransport(GelfTransports.TCP);
        config.setHost("127.0.0.1");
        config.setPort(12201);

        // Optional settings
        config.setConnectTimeout(5000);
        config.setReconnectDelay(1000);
        config.setTcpNoDelay(true);
        config.setQueueSize(512);
        config.setSendBufferSize(32768);

        GelfMessageBuilder builder = new GelfMessageBuilder(GelfMessageVersion.V1_1);
        GelfMessage messageTemplate = builder.addHost("localhost")
                                        .addAdditionalField("_foo", "bar");

        GelfTransport transport = GelfTransports.create(config);

        for (int i = 0; i < 100; i++) {
            GelfMessage message = messageTemplate.build();

            message.addMessage("This is message #" + i);
            message.addAdditionalField("_count", i);

            if (blocking) {
                // Blocks until there is capacity in the queue
                transport.send(message);
            } else {
                // Returns false if there isn't enough room in the queue
                boolean enqueued = transport.trySend(message);
            }
        }

        transport.stop();
    }
}
```

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## License

Apache License, Version 2.0 -- http://www.apache.org/licenses/LICENSE-2.0
