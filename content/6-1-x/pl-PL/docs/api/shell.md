# shell

> Manage files and URLs using their default applications.

Proces: [Main](../glossary.md#main-process), [Renderer](../glossary.md#renderer-process)

The `shell` module provides functions related to desktop integration.

An example of opening a URL in the user's default browser:

```javascript
const { shell } = require('electron')

shell.openExternal('https://github.com')
```

## Metody

The `shell` module has the following methods:

### `shell.showItemInFolder(fullPath)`

* `fullPath` String

Show the given file in a file manager. If possible, select the file.

### `shell.openItem(fullPath)`

* `fullPath` String

Returns `Boolean` - Whether the item was successfully opened.

Otwiera dany plik w normalnym sposobie komputera.

### `shell.openExternalSync(url[, options])`

* `url` String - Max 2081 characters on Windows, or the function returns false.
* `options` Object (optional)
  * `activate` Boolean (optional) - `true` to bring the opened application to the foreground. Domyślnie jest `true`. _macOS_
  * `workingDirectory` String (optional) - The working directory. _Windows_

Returns `Boolean` - Whether an application was available to open the URL.

Open the given external protocol URL in the desktop's default manner. (For example, mailto: URLs in the user's default mail agent).

**Przestarzałe**

### `shell.openExternal(url[, options])`

* `url` String - Max 2081 characters on windows.
* `options` Object (optional)
  * `activate` Boolean (optional) - `true` to bring the opened application to the foreground. Domyślnie jest `true`. _macOS_
  * `workingDirectory` String (optional) - The working directory. _Windows_

Zwraca `Promise<void>`

Open the given external protocol URL in the desktop's default manner. (For example, mailto: URLs in the user's default mail agent).

### `shell.moveItemToTrash(fullPath)`

* `fullPath` String

Returns `Boolean` - Whether the item was successfully moved to the trash.

Move the given file to trash and returns a boolean status for the operation.

### `shell.beep()`

Play the beep sound.

### `shell.writeShortcutLink(shortcutPath[, operation], options)` _Windows_

* `shortcutPath` String
* `operation` String (optional) - Default is `create`, can be one of following:
  * `create` - Creates a new shortcut, overwriting if necessary.
  * `update` - Updates specified properties only on an existing shortcut.
  * `replace` - Overwrites an existing shortcut, fails if the shortcut doesn't exist.
* `options` [ShortcutDetails](structures/shortcut-details.md)

Zwraca `Boolean` - Określa, czy skrót został utworzony pomyślnie.

Creates or updates a shortcut link at `shortcutPath`.

### `shell.readShortcutLink(shortcutPath)` _Windows_

* `shortcutPath` String

Zwraca [`ShortcutDetails`](structures/shortcut-details.md)

Resolves the shortcut link at `shortcutPath`.

An exception will be thrown when any error happens.
