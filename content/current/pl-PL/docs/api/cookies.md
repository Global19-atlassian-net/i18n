## Klasa: Cookies

> Query and modify a session's cookies.

Proces: [Main](../glossary.md#main-process)

Instances of the `Cookies` class are accessed by using `cookies` property of a `Session`.

Na przykład:

```javascript
const { session } = require('electron')

// Query all cookies.
session.defaultSession.cookies.get({})
  .then((cookies) => {
    console.log(cookies)
  }).catch((error) => {
    console.log(error)
  })

// Query all cookies associated with a specific url.
session.defaultSession.cookies.get({ url: 'http://www.github.com' })
  .then((cookies) => {
    console.log(cookies)
  }).catch((error) => {
    console.log(error)
  })

// Set a cookie with the given cookie data;
// may overwrite equivalent cookies if they exist.
const cookie = { url: 'http://www.github.com', name: 'dummy_name', value: 'dummy' }
session.defaultSession.cookies.set(cookie)
  .then(() => {
    // success
  }, (error) => {
    console.error(error)
  })
```

### Wydarzenia instancji

The following events are available on instances of `Cookies`:

#### Zdarzenie: 'changed'

* `event` Event
* `cookie` [Cookie](structures/cookie.md) - The cookie that was changed.
* `cause` String - The cause of the change with one of the following values:
  * `explicit` - The cookie was changed directly by a consumer's action.
  * `overwrite` - The cookie was automatically removed due to an insert operation that overwrote it.
  * `expired` - The cookie was automatically removed as it expired.
  * `evicted` - The cookie was automatically evicted during garbage collection.
  * `expired-overwrite` - The cookie was overwritten with an already-expired expiration date.
* `removed` Boolean - `true` if the cookie was removed, `false` otherwise.

Emitted when a cookie is changed because it was added, edited, removed, or expired.

### Metody instancji

The following methods are available on instances of `Cookies`:

#### `cookies.get(filter)`

* `filter` Object
  * `url` String (optional) - Retrieves cookies which are associated with `url`. Empty implies retrieving cookies of all URLs.
  * `name` String (optional) - Filters cookies by name.
  * `domain` String (optional) - Retrieves cookies whose domains match or are subdomains of `domains`.
  * `path` String (optional) - Retrieves cookies whose path matches `path`.
  * `secure` Boolean (optional) - Filters cookies by their Secure property.
  * `session` Boolean (optional) - Filters out session or persistent cookies.

Returns `Promise<Cookie[]>` - A promise which resolves an array of cookie objects.

Sends a request to get all cookies matching `filter`, and resolves a promise with the response.

#### `cookies.set(details)`

* `details` Object
  * `url` String - The URL to associate the cookie with. The promise will be rejected if the URL is invalid.
  * `name` String (optional) - The name of the cookie. Empty by default if omitted.
  * `value` String (optional) - The value of the cookie. Empty by default if omitted.
  * `domain` String (opcjonalne) - Domena ciasteczka; wartość zostanie znormalizowane poprzez dodanie kropki na początku, przez co będzie również działać z subdomenami. Empty by default if omitted.
  * `path` String (opcjonalne) - Ścieżka ciasteczka. Empty by default if omitted.
  * `secure` Boolean (optional) - Whether the cookie should be marked as Secure. Defaults to false.
  * `httpOnly` Boolean (optional) - Whether the cookie should be marked as HTTP only. Defaults to false.
  * `expirationDate` Double (optional) - The expiration date of the cookie as the number of seconds since the UNIX epoch. If omitted then the cookie becomes a session cookie and will not be retained between sessions.

Returns `Promise<void>` - A promise which resolves when the cookie has been set

Sets a cookie with `details`.

#### `cookies.remove(url, name)`

* `url` String - The URL associated with the cookie.
* `name` String - The name of cookie to remove.

Returns `Promise<void>` - A promise which resolves when the cookie has been removed

Removes the cookies matching `url` and `name`

#### `cookies.flushStore()`

Returns `Promise<void>` - A promise which resolves when the cookie store has been flushed

Writes any unwritten cookies data to disk.
