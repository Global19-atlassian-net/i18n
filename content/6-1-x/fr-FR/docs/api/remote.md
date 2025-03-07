# remote

> Utilise les modules du processus main depuis le processus renderer.

Processus : [Renderer](../glossary.md#renderer-process)

Le module `distant` fournit un moyen simple de faire une communication entre les processus d'inter-processus (IPC) entre le processus de rendu (page Web) et le processus principal.

Dans Electron, les modules liés à l'interface graphique (comme `dialogue`, `menu` etc.) ne sont disponibles que dans le processus principal, pas dans le processus de rendu. Afin de les utiliser depuis le processus de rendu, le module `ipc` est nécessaire pour envoyer des messages inter-processus au processus principal. Avec le module `distant`, vous pouvez appeler les méthodes de l'objet principal du processus sans envoyer explicitement des messages inter-processus, similaires à la [RMI](https://en.wikipedia.org/wiki/Java_remote_method_invocation)de Java. Un exemple de création d'une fenêtre de navigateur à partir d'un processus de rendu :

```javascript
const { BrowserWindow } = require('electron').remote
let win = new BrowserWindow({ width: 800, height: 600 })
win.loadURL('https://github.com')
```

**Note:** For the reverse (access the renderer process from the main process), you can use [webContents.executeJavaScript](web-contents.md#contentsexecutejavascriptcode-usergesture-callback).

**Remarque :** Le module distant peut être désactivé pour des raisons de sécurité dans les contextes suivants :
- [`BrowserWindow`](browser-window.md) - en définissant l'option `enableRemoteModule` à `false`.
- [`<webview>`](webview-tag.md)`</0> - en définissant l'attribut <2>enableremotemodule</2> à <2>false</2>.</li>
</ul>

<h2 spaces-before="0">Objet remote</h2>

<p spaces-before="0">Chaque objet (y compris les fonctions) retourné par le module <code>distance` représente un objet dans le processus principal (nous l'appelons un objet distant ou une fonction distante). Lorsque vous appelez des méthodes d'un objet distant, appelez une fonction distante, ou créez un nouvel objet avec le constructeur distant (fonction), vous envoyez en fait messages inter-processus synchrones.</p>

Dans l'exemple ci-dessus, [`BrowserWindow`](browser-window.md) et `win` étaient des objets distants et `new BrowserWindow` n'a pas créé d'objet `BrowserWindow` dans le processus de rendu . Au lieu de cela, il a créé un objet `BrowserWindow` dans le processus principal et a renvoyé l'objet distant correspondant dans le processus de rendu, soit l'objet `win`.

**Remarque :** Seules les [propriétés énumérables](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties) présentes lorsque l'objet distant est référencé pour la première fois sont accessibles par télécommande.

**Remarque :** Les tableaux et les tampons sont copiés par IPC lorsque vous y accédez via le module `distance` . Les modifier dans le processus de rendu ne les modifie pas dans le processus principal et vice versa.

## Durée de vie de l'objet remote

Electron s'assure que tant que l'objet distant dans le processus de rendu vit (en d'autres mots, n'a pas été ramassé), l'objet correspondant dans le processus principal ne sera pas libéré. Lorsque l'objet distant a été collecté , l'objet correspondant dans le processus principal sera déférencé par .

Si l'objet distant est divulgué dans le processus de rendu (par ex. stocké dans une carte mais jamais <unk> ), l'objet correspondant dans le processus principal sera également divulgué, donc vous devriez être très prudent de ne pas fuir des objets distants.

Les types de valeurs primaires comme les chaînes de caractères et les nombres, sont envoyés par copie.

## Passage des callbacks au processus principal

Le code dans le processus principal peut accepter les callbacks du moteur de rendu - par exemple le module `distant` - mais vous devriez être extrêmement prudent lorsque vous utilisez cette fonctionnalité .

First, in order to avoid deadlocks, the callbacks passed to the main process are called asynchronously. You should not expect the main process to get the return value of the passed callbacks.

Par exemple, vous ne pouvez pas utiliser une fonction du processus de rendu dans un `Array.map` appelé dans le processus principal :

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
// renderer process
const mapNumbers = require('electron').remote.require('./mapNumbers')
const withRendererCb = mapNumbers.withRendererCallback(x => x + 1)
const withLocalCb = mapNumbers.withLocalCallback()

console.log(withRendererCb, withLocalCb)
// [undefined, undefined, undefined], [2, 3, 4]
```

Comme vous pouvez le voir, la valeur synchrone de la fonction de rappel du moteur de rendu n'était pas comme attendu, et ne correspond pas à la valeur de retour d'un callback identique qui vit dans le processus principal.

Deuxièmement, les callbacks passés au processus principal persisteront jusqu'à ce que le processus principal les ramasse.

For example, the following code seems innocent at first glance. It installs a callback for the `close` event on a remote object:

```javascript
require('electron').remote.getCurrentWindow().on('close', () => {
  // window was closed...
})
```

Mais n'oubliez pas que le callback est référencé par le processus principal jusqu'à ce que vous le désinstallez explicitement explicitement. Si vous ne le faites pas, chaque fois que vous rechargez votre fenêtre, la fonction de rappel sera réinstallée, fuyant un rappel pour chaque redémarrage.

Pour empirer les choses, puisque le contexte des callbacks précédemment installés a été libéré, les exceptions seront levées dans le processus principal lorsque l'événement `close` est émis.

Pour éviter ce problème, assurez-vous de nettoyer toutes les références pour rendre les callbacks passés au processus principal. Cela implique de nettoyer les gestionnaires d'événements, ou en s'assurant que le processus principal est explicitement dit de déréférencer les callbacks qui sont venus d'un processus de rendu qui se termine.

## Accès aux modules intégrés dans le processus principal

Les modules intégrés dans le processus principal sont ajoutés en tant que récupérateurs dans le module `distance` , ainsi vous pouvez les utiliser directement comme le module `electron`.

```javascript
const app = require('electron').remote.app
console.log(app)
```

## Méthodes

Le module `remote` dispose des méthodes suivantes :

### `remote.require(module)`

* `module` String

Retourne `any` - L'objet retourné par `require(module)` dans le processus principal. Les modules spécifiés par leur chemin relatif résoudront par rapport au point d'entrée du processus principal.

exemple :

```sh
project/
├── main
│   ├── foo.js
│   └── index.js
├── package.json
└── renderer
    └── index.js
```

```js
// main process: main/index.js
const { app } = require('electron')
app.on('ready', () => { /* ... */ })
```

```js
// un module relatif: main/foo.js
module.exports = 'bar'
```

```js
// renderer process: renderer/index.js
const foo = require('electron').remote.require('./foo') // bar
```

### `remote.getCurrentWindow()`

Retourne [`BrowserWindow`](browser-window.md) - La fenêtre à laquelle cette page web appartient.

**Remarque :** N'utilisez pas `removeAllListeners` sur [`BrowserWindow`](browser-window.md). L'utilisation de ceci peut supprimer tous les auditeurs [`flou`](https://developer.mozilla.org/en-US/docs/Web/Events/blur) , désactiver les événements de clic sur les boutons de la barre tactile, et d'autres conséquences involontaires.

### `remote.getCurrentWebContents()`

Retourne [`WebContents`](web-contents.md) - Le contenu web de cette page web.

### `remote.getGlobal(name)`

* `name` String

Retourne `any` - La variable globale de `name` (par exemple `global[name]`) dans le processus principal.

## Propriétés

### `remote.process`

The `process` object in the main process. This is the same as `remote.getGlobal('process')` but is cached.
