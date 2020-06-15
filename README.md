# Fastify SSE Plugin
![CI check](https://github.com/NodeFactoryIo/fastify-sse-v2/workflows/CI%20check/badge.svg?branch=master)
[![npm version](https://badge.fury.io/js/fastify-sse-v2.svg)](https://badge.fury.io/js/fastify-sse-v2)

Fastify plugin for sending [Server-sent events](https://en.wikipedia.org/wiki/Server-sent_events).

### How to use?

```terminal
yarn add fastify-sse-v2
```
Register fastify-sse-v2 plugin into your fastify instance:
```javascript
import {FastifySSEPlugin} from "fastify-sse-v2";

const server = fastify();
server.register(FastifySSEPlugin);
```

#### Sending events from AsyncIterable source

```javascript
import {FastifySSEPlugin} from "fastify-sse-v2";

const server = fastify();
server.register(FastifySSEPlugin);

server.get("/", function (req, res) {
    res.sse((async function * source () {
          for (let i = 0; i < 10; i++) {
            sleep(2000);
            yield {id: String(i), data: "Some message"};
          }
    })());
});
```

##### Sending events from EventEmmiters

```javascript
import {FastifySSEPlugin} from "fastify-sse-v2";
import EventIterator from "event-iterator";

const server = fastify();
server.register(FastifySSEPlugin);

server.get("/", function (req, res) {
    const eventEmitter = new EventEmitter();
    res.sse(new EventIterator(
                (push) => {
                  eventEmitter.on("some_event", push)
                  return () => eventEmitter.removeEventListener("some_event", push)
                }
        )
    );
});
```

##### Note
- to remove event listeners (or some other cleanup) when client closes connection,
 you can listen on fastify's [req.raw connection close event](https://nodejs.org/docs/latest-v12.x/api/http.html#http_event_close_2):

