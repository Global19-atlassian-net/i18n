# ang remote

> Gamitin ang mga modyul ng pangunahing proseso mula sa proseso ng tagabigay.

Mga proseso: [Renderer](../glossary.md#renderer-process)

Ang modyul ng `remote` ay nagbibigay ng isang simpleng paraan na gumawa ng maki-prosesong komyunikasyon (IPC) sa pagitan ng proaesong tagabigay (pahina ng web) at ng pangunahing proseso.

Sa Electron, ang mga modyul na may kaugnayan sa GUI (tulad ng `dialog`, `menu` atbp.) ay makukuha lamang sa pangunahing proseso, hindi sa prosesong tagabigay. Sa ayos ng paggamit sa kanila mula sa prosesong tagabigay, ang modyul ng `ipc` ay kinakailangan na magpadala ng mga maki-prosesong mensahe sa pangunahing proseso. Kasama ang modyul ng `remote`, maaari mong hingiin ang mga pamamaraan ng mga bagay sa pangunahing proseso nang walang nakakapansin na nakapagpadala ng maki-prosesong mensahe, katulad ng [RMI](https://en.wikipedia.org/wiki/Java_remote_method_invocation) ng Java. Ang isang halimbawa ng paglikha ng isang browser window mula sa isang prosesong tagabigay:

```javascript
const { BrowserWindow } = kailangan('electron').remote
let win = new BrowserWindow({ width: 800, height: 600 })
win.loadURL('https://github.com')
```

**Note:** For the reverse (access the renderer process from the main process), you can use [webContents.executeJavaScript](web-contents.md#contentsexecutejavascriptcode-usergesture).

**Note:** The remote module can be disabled for security reasons in the following contexts:
- [`BrowserWindow`](browser-window.md) - by setting the `enableRemoteModule` option to `false`.
- [`<webview>`](webview-tag.md) - by setting the `enableremotemodule` attribute to `false`.

## Mga bagay ng Remote

Bawat bagay (kasama ang mga punsyon) ay nagbabalik dahil ang modyul ng `remote` ay kumakatawan sa isang bagay sa pangunahing proseso (tinatawag natin itong malayong bagay o malayong punsyon). Kapag iyong hiningi ang mga pamamaraan ng isang remote na bagay, tawagin ang isang remote na punsyon, o gumawa ng isang bagong bagay kasama ang remote na tagagawa (punsyon), ikaw ay talagang nagpapadala ng sabay-sabay na mga maki-prosesong mensahe.

In the example above, both [`BrowserWindow`](browser-window.md) and `win` were remote objects and `new BrowserWindow` didn't create a `BrowserWindow` object in the renderer process. Sa halip, ito ay gumawa ng isang bagay ng `BrowserWindow` sa pangunahing proseso at ibinalik ang nararapat na remote na bagay sa prosesong tagabigay, kagaya ng bagay sa `win`.

**Note:** Ang [enurable properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties) lamang na kung saan ay naroroon nang ang remote na bagay ay unang isinangguni na mapupuntahan sa pamamagitan ng remote.

**Note:** Ang mga hanay at mga Buffer ay kinopya sa ibabaw ng IPC kapag na-access sa pamamagitan ng modyul ng `remote`. Ang pagbabago sa kanila sa prosesong tagabigay ay hindi magpapabago sa kanila sa pangunahing proseso at kahit sa kabaligtaran.

## Ang tagal ng buhay ng mga Remote na Bagay

Sinisigurado ng Electron na habang ang remote na bagay ay nandoon pa sa prosesong tagabigay (sa ibang salita, ay hindi pa ibinabasura), ang nararapat na bagay sa pangunahing proseso ay hindi mailalabas. Kapag ang remote na bagay ay naibasura na, ang nararapat na bagay sa pangunahing proseso ay hindi isasangguni.

Kung ang remote na bagay ay nakalabas sa prosesong tagabigay (hal. itinago sa isang balangkas ngunit hindi nailabas), ang nararapat na bagay sa pangunahing proseso ay makakalabas din, kaya kailangan mo ng matinding pag-iingat na hindi makalabas ang mga remote na bagay.

Ang mga uri ng pangunahing halaga tulad ng mga string at mga numero, gayunpaman, ay ipinapadala sa pamamagitan ng pagkopya.

## Ipinapasa ang gantingtawag patungo sa pangunahing proseso

Ang kodigo sa pangunahing proaeso ay maaaring tanggapin ang mga gantingtawag mula sa tagabigay - sa isang pagkakataon ang modyul ng `remote` - ngunit kailangan mo ng matinding pag-iingat kapag ginagamit ang katangian na ito.

First, in order to avoid deadlocks, the callbacks passed to the main process are called asynchronously. You should not expect the main process to get the return value of the passed callbacks.

Sa isang pagkakataon hindi mo magagamit ang isang punsyon mula sa prosesong tagabigay sa isang `Array.map` na tinawag sa pangunahing proseso:

```javascript
// main process mapNumbers.js
exports.withRendererCallback = (mapper) => {
  return [1, 2, 3].map(mapper)
}

exports.withLocalCallback = () => {
  return [1, 2, 3].map(x => x + 1)
}
```

```javascript
// proseso ng tagabigay
const mapNumbers = kailangan('electron').remote.require('./mapNumbers')
const withRendererCb = mapNumbers.withRendererCallback(x => x + 1)
const withLocalCb = mapNumbers.withLocalCallback()

console.log(withRendererCb, withLocalCb)
// [undefined, undefined, undefined], [2, 3, 4]
```

Kagaya ng iyong nakikita, ang magkakasabay na nagbalik na halaga ng gantingtawag ng tagabigay ay hindi inaasahan, at hindi tumutugma ang nagbalik na halaga ng isang kilalang gantingtawag na naroroon sa pangunahing proseso.

Pangalawa, ang mga gantingtawag na ipinasa sa pangunahing proseso ay mananatili hanggang sila ay ibasura ng pangunahing proseso.

For example, the following code seems innocent at first glance. It installs a callback for the `close` event on a remote object:

```javascript
require('electron').remote.getCurrentWindow().on('close', () => {
  // window was closed...
})
```

Ngunit tandaan ang gantingtawag ay isinangguni nang pangunahing proseso hanggang tahasan mong ito ay i-uninstall. Kung hindi mo, sa bawat pag-reload mo ng iyong window ang gantingtawag ay iiinstall ulit, lalabas ang isang gantingtawag sa bawat restart.

Ang mas masahol pa, matapos na ang konteksto ng dati ng in-install na mga gantingtawag ay nailabas na, ang mga eksepsyon ay itinaas na sa pangunahing proseso kapag ang event ng `close` ay lumabas na.

Upang maiwasan ang problema, siguraduhin burahin anumang kaugnayan sa mga binalikang tawag na ipinasa sa pangunahing proseso. This involves cleaning up event handlers, or ensuring the main process is explicitly told to dereference callbacks that came from a renderer process that is exiting.

## Pag-access ng mga built-in na modyul sa mga pangunahing proseso

Ang mga built-in na modyul sa mga pangunahing proseso ay idinadagdag bilang mga tagakuha sa modyul ng `remote`, kaya maaari mo silang gamitin ng direkta katulad ng modyul ng `electron`.

```javascript
const app = kailangan('electron').remote.app
console.log(app)
```

## Mga Paraan

Ang modyul ng `remote` ay mayroon ng mga sumusunod na pamamaraan:

### `remote.require(modyul)`

* `modyul` Lubid

Nagbabalik ang `any` - Ang bagay ay nagbalik sa pamamagitan ng `require(module)` sa mga pangunahing proseso. Ang mga modyul na tinukoy nang kanilang kaugnay na landas ay malulutas ng may kaugnayan sa mga pasukan ng mga pangunahing proseso.

halimbawa.

```sh
proyekto/
├── pangunahing
│   ├── foo.js
│   └── index.js
├── package.json
└── tagabigay
    └── index.js
```

```js
// main process: main/index.js
const { app } = kailangan('electron')
app.on('ready', () => { /* ... */ })
```

```js
// ilang kaugnay na modyul: main/foo.js
module.exports = 'bar'
```

```js
// prosesong tagasalin: renderer/index.js
const foo = kailangan('electron').remote.require('./foo') // bar
```

### `remote.getCurrentWindow()`

Nagbabalik ang [`BrowserWindow`](browser-window.md) - Ang window na kung saan nabibilang ang pahina ng web na ito.

**Note:** Do not use `removeAllListeners` on [`BrowserWindow`](browser-window.md). Use of this can remove all [`blur`](https://developer.mozilla.org/en-US/docs/Web/Events/blur) listeners, disable click events on touch bar buttons, and other unintended consequences.

### `remote.getCurrentWebContents()`

Nagbabalik ang [`WebContents`](web-contents.md) - Ang mga laman ng web ng pahina ng web na ito.

### `remote.getGlobal(pangalan)`

* `name` String

Nagbabalik ang `any` - Ang global na pagbabago-bago ng `name` (hal. `global[name]`) sa mga pangunahing proseso.

## Mga Katangian

### `remote.process` _Readonly_

A `NodeJS.Process` object.  The `process` object in the main process. This is the same as `remote.getGlobal('process')` but is cached.
