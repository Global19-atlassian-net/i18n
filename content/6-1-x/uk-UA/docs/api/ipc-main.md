# ipcMain

> Communicate asynchronously from the main process to renderer processes.

Процес: [Main](../glossary.md#main-process)

The `ipcMain` module is an instance of the [EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter) class. When used in the main process, it handles asynchronous and synchronous messages sent from a renderer process (web page). Messages sent from a renderer will be emitted to this module.

## Sending Messages

It is also possible to send messages from the main process to the renderer process, see [webContents.send](web-contents.md#contentssendchannel-arg1-arg2-) for more information.

* When sending a message, the event name is the `channel`.
* To reply to a synchronous message, you need to set `event.returnValue`.
* To send an asynchronous message back to the sender, you can use `event.reply(...)`.  This helper method will automatically handle messages coming from frames that aren't the main frame (e.g. iframes) whereas `event.sender.send(...)` will always send to the main frame.

An example of sending and handling messages between the render and main processes:

```javascript
// In main process.
const { ipcMain } = require('electron')
ipcMain.on('asynchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.reply('asynchronous-reply', 'pong')
})

ipcMain.on('synchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.returnValue = 'pong'
})
```

```javascript
// In renderer process (web page).
const { ipcRenderer } = require('electron')
console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // prints "pong"

ipcRenderer.on('asynchronous-reply', (event, arg) => {
  console.log(arg) // prints "pong"
})
ipcRenderer.send('asynchronous-message', 'ping')
```

## Методи

The `ipcMain` module has the following method to listen for events:

### `ipcMain.on(channel, listener)`

* `channel` String
* `listener` Function
  * `event` IpcMainEvent
  * `...args` any[]

Listens to `channel`, when a new message arrives `listener` would be called with `listener(event, args...)`.

### `ipcMain.once(channel, listener)`

* `channel` String
* `listener` Function
  * `event` IpcMainEvent
  * `...args` any[]

Adds a one time `listener` function for the event. This `listener` is invoked only the next time a message is sent to `channel`, after which it is removed.

### `ipcMain.removeListener(channel, listener)`

* `channel` String
* `listener` Function

Removes the specified `listener` from the listener array for the specified `channel`.

### `ipcMain.removeAllListeners([channel])`

* `channel` String

Removes listeners of the specified `channel`.

## Event object

The documentation for the `event` object passed to the `callback` can be found in the [`ipc-main-event`](structures/ipc-main-event.md) structure docs.