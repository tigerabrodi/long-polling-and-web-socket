# Long Polling

- Simplest way of having persistent connection with server, without using Web Sockets.

- Regular polling comes with downsides, delayed responses and if there are no new information from the server, it is still bombed with requests every 10 seconds.

- Long Polling is easy to implement and also is a better way to poll the server:

1. A request is sent to the server.

2. The server doesn't close the connection until it has a message to send.

3. When a message appears, the server responds to the request with it.

4. The browser makes a new request immediately.

- If the connection is lost for some reason, the browser immediately sends a new request.

A sketch of client-side subscribe function that has long polling implemented:

```js
async function subscribe() {
  let response = await fetch("/subscribe");

  if (response.status == 502) {
    // Status 502 is a connection timeout error,
    // may happen when the connection was pending for too long,
    // and the remote server or a proxy closed it
    // let's reconnect
    await subscribe();
  } else if (response.status != 200) {
    // An error - let's show it
    showMessage(response.statusText);
    // Reconnect in one second
    await new Promise(resolve => setTimeout(resolve, 1000));
    await subscribe();
  } else {
    // Get and show the message
    let message = await response.text();
    showMessage(message);
    // Call subscribe() again to get the next message
    await subscribe();
  }
}

subscribe();
```

# Web Socket

- Provides a way to exchange data between browser and server via a persistent connection.

- Data can be passed in both directions without breaking the connection nor having to send new HTTP requests.

- To start a connection, we need to create it using `new WebSocket("ws://....")`.

- When starting a connection, we need to use a special protocol in the beginning of a URL: `ws`.

- There is also a more secure and reliable protocol which is encrypted, instead of `ws`, `wss`. This is kinda similar to `http` vs `https`.

- Once the socket is created, a connection is established, we can listen to 4 events: open, message, error and close.

- To send something we can use the `send` method: `socket.send(data)`.

- To close the connection you can use `socket.close(code, reason)`. The `code` and `reason` can be accessed during listening on the close event of the other party: `event.code`, `event.reason`. Full list of codes and their meaning when a connection gets closed: https://tools.ietf.org/html/rfc6455#section-7.4.1.

- Rate limiting: If keep sending a lot of data and the user is on slower connection, the data will be buffered (stored) in memory. `socket.bufferedAmount` can be used to check if the socket is available for transmission, meaning we can send data.

- To check the connection's state, there is `socket.readyState`.
