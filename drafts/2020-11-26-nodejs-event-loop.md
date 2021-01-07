current -> nextTick -> setTimeout(()=>{}, 0) -> next event loop iteration

setImmediate vs setInterval

the order in which the two timers are executed is non-deterministic,
if you move the two calls within an I/O cycle, the immediate callback is always executed first

[next tick](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
[why use](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#why-use-process-nexttick)

```javascript
const server = net.createServer(() => {}).listen(8080);

server.on("listening", () => {});
```

When only a port is passed, the port is bound immediately. So, the 'listening' callback could be called immediately. The problem is that the .on('listening') callback will not have been set by that time.

To get around this, the 'listening' event is queued in a nextTick() to allow the script to run to completion. This allows the user to set any event handlers they want.
