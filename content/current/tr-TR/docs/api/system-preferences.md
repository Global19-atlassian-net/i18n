# systemPreferences

> Sistem tercihlerini al.

İşlem: [Ana](../glossary.md#main-process)

```javascript
const { systemPreferences } = require('electron')
console.log(systemPreferences.isDarkMode())
```

## Etkinlikler

`systemPreferences` nesnesi aşağıdaki olayları yayar:

### Event: 'accent-color-changed' _Windows_

Dönüşler:

* `event` Olay
* `newColor` String - Kullanıcının sistemine atadığı yeni RGBA vurgu rengi.

### Event: 'color-changed' _Windows_

Dönüşler:

* `olay` Olay

### Event: 'inverted-color-scheme-changed' _Windows_ _Deprecated_

Dönüşler:

* `event` Event
* `invertedColorScheme` Boolean - `true` if an inverted color scheme (a high contrast color scheme with light text and dark backgrounds) is being used, `false` otherwise.

**Deprecated:** Should use the new [`updated`](native-theme.md#event-updated) event on the `nativeTheme` module.

### Event: 'high-contrast-color-scheme-changed' _Windows_ _Deprecated_

Döndürür:

* `event` Olay
* `highContrastColorScheme` Boolean - `true` if a high contrast theme is being used, `false` otherwise.

**Deprecated:** Should use the new [`updated`](native-theme.md#event-updated) event on the `nativeTheme` module.

## Metodlar

### `systemPreferences.isDarkMode()` _macOS_ _Windows_ _Deprecated_

Sistemin karanlık modda olup olmadığına dair `Boolean` döndürür.

**Note:** On macOS 10.15 Catalina in order for this API to return the correct value when in the "automatic" dark mode setting you must either have `NSRequiresAquaSystemAppearance=false` in your `Info.plist` or be on Electron `>=7.0.0`.  See the [dark mode guide](../tutorial/mojave-dark-mode-guide.md) for more information.

**Deprecated:** Should use the new [`nativeTheme.shouldUseDarkColors`](native-theme.md#nativethemeshouldusedarkcolors-readonly) API.

### `systemPreferences.isSwipeTrackingFromScrollEventsEnabled()` _macOS_

Sayfalar arasında kaydırma ayarı açık olup olmadığına dair `Boolean` döndürür.

### `systemPreferences.postNotification(event, userInfo[, deliverImmediately])` _macOS_

* `event` String
* `userInfo` Record<String, any>
* `deliverImmediately` Boolean (optional) - `true` to post notifications immediately even when the subscribing app is inactive.

Posts `event` as native notifications of macOS. The `userInfo` is an Object that contains the user information dictionary sent along with the notification.

### `systemPreferences.postLocalNotification(event, userInfo)` _macOS_

* `event` String
* `userInfo` Record<String, any>

Posts `event` as native notifications of macOS. The `userInfo` is an Object that contains the user information dictionary sent along with the notification.

### `systemPreferences.postWorkspaceNotification(event, userInfo)` _macOS_

* `event` String
* `userInfo` Record<String, any>

Posts `event` as native notifications of macOS. The `userInfo` is an Object that contains the user information dictionary sent along with the notification.

### `systemPreferences.subscribeNotification(event, callback)` _macOS_

* `event` String
* `callback` Function
  * `event` String
  * `userInfo` Record<String, unknown>
  * `object` String

Returns `Number` - The ID of this subscription

İlgili `event` gerçekleştiğinde MacOS'un yerel bildirimlerine abone olup `callback`, `callback(event, userInfo)` ile beraber çağırılmış olacak. `userInfo` bildirim ile birlikte gönderilen kullanıcı bilgileri sözlüğünü içeren bir objedir. The `object` is the sender of the notification, and only supports `NSString` values for now.

`event`'ın aboneliğini iptal etmek için kullanılabilecek abonenin `id`'sini döndürür.

Bu başlığının altında API `NSDistributedNotificationCenter`'e abone olur, `event`'ın örnek değerleri şöyledir:

* `AppleInterfaceThemeChangedNotification`
* `AppleAquaColorVariantChanged`
* `AppleColorPreferencesChangedNotification`
* `AppleShowScrollBarsSettingChanged`

### `systemPreferences.subscribeLocalNotification(event, callback)` _macOS_

* `event` String
* `callback` Function
  * `event` String
  * `userInfo` Record<String, unknown>
  * `object` String

Returns `Number` - The ID of this subscription

Same as `subscribeNotification`, but uses `NSNotificationCenter` for local defaults. This is necessary for events such as `NSUserDefaultsDidChangeNotification`.

### `systemPreferences.subscribeWorkspaceNotification(event, callback)` _macOS_

* `event` String
* `callback` Function
  * `event` String
  * `userInfo` Record<String, unknown>
  * `object` String

Same as `subscribeNotification`, but uses `NSWorkspace.sharedWorkspace.notificationCenter`. This is necessary for events such as `NSWorkspaceDidActivateApplicationNotification`.

### `systemPreferences.unsubscribeNotification(id)` _macOS_

* `id` tamsayı

Aboneyi `id` ile kaldırır.

### `systemPreferences.unsubscribeLocalNotification(id)` _macOS_

* `id` tamsayı

`unsubscribeNotification` gibidir fakat aboneyi `NSNotificationCenter`'den çıkarır.

### `systemPreferences.unsubscribeWorkspaceNotification(id)` _macOS_

* `id` tamsayı

Same as `unsubscribeNotification`, but removes the subscriber from `NSWorkspace.sharedWorkspace.notificationCenter`.

### `systemPreferences.registerDefaults(defaults)` _macOS_

* `defaults` Record<String, String | Boolean | Number> - a dictionary of (`key: value`) user defaults

Add the specified defaults to your application's `NSUserDefaults`.

### `systemPreferences.getUserDefault(key, type)` _macOS_

* `key` String
* `type` String - Can be `string`, `boolean`, `integer`, `float`, `double`, `url`, `array` or `dictionary`.

`any` - `NSUserDefaults` 'te `key` değerini verir.

Bazı popüler `key` ve `type`'ler:

* `AppleInterfaceStyle`: `string`
* `AppleAquaColorVariant`: `integer`
* `AppleHighlightColor`: `string`
* `AppleShowScrollBars`: `string`
* `NSNavRecentPlaces`: `array`
* `NSPreferredWebServices`: `dictionary`
* `NSUserDictionaryReplacementItems`: `array`

### `systemPreferences.setUserDefault(key, type, value)` _macOS_

* `key` String
* `type` String - See [`getUserDefault`](#systempreferencesgetuserdefaultkey-type-macos).
* `value` String

`NSUserDefaults`'de `key` değerini ayarlayın.

Note that `type` should match actual type of `value`. An exception is thrown if they don't.

Bazı popüler `key` ve `type`'ler:

* `ApplePressAndHoldEnabled`: `boolean`

### `systemPreferences.removeUserDefault(key)` _macOS_

* `key` String

Removes the `key` in `NSUserDefaults`. This can be used to restore the default or global value of a `key` previously set with `setUserDefault`.

### `systemPreferences.isAeroGlassEnabled()` _Windows_

Eğer [DWM composition](https://msdn.microsoft.com/en-us/library/windows/desktop/aa969540.aspx) (Aero Glass) aktifse `true` aksi takdirde `false` `Boolean` değerini döndürecektir.

Şeffaf bir pencere oluşturmanız gerekip gerekmediğini belirlemek için bunu kullanıp kullanmamanın bir örneği (DWM kompozisyonu devre dışı bırakıldığında şeffaf pencere düzgün çalışmaz):

```javascript
const { BrowserWindow, systemPreferences } = require('electron')
let browserOptions = { width: 1000, height: 800 }

// Pencereyi sadece platform destekliyorsa şeffaf yapın.
if (process.platform !== 'win32' || systemPreferences.isAeroGlassEnabled()) {
  browserOptions.transparent = true
  browserOptions.frame = false
}

// Pencere yaratın.
let win = new BrowserWindow(browserOptions)

// Yönlendirme.
if (browserOptions.transparent) {
  win.loadURL(`file://${__dirname}/index.html`)
} else {
  // Şeffaflık yok, bu nedenle temel stilleri kullanan bir geri yükleme yüklüyoruz.
  win.loadURL(`file://${__dirname}/fallback.html`)
}
```

### `systemPreferences.getAccentColor()` _Windows_ _macOS_

RGBA'da onaltılık formda kullanıcıların mevcut sistemindeki geniş vurgulu renk tercihini `String` olarak döndürür.

```js
const color = systemPreferences.getAccentColor() // `"aabbccdd"`
const red = color.substr(0, 2) // "aa"
const green = color.substr(2, 2) // "bb"
const blue = color.substr(4, 2) // "cc"
const alpha = color.substr(6, 2) // "dd"
```

This API is only available on macOS 10.14 Mojave or newer.

### `systemPreferences.getColor(color)` _Windows_ _macOS_

* `color` String - One of the following values:
  * On **Windows**:
    * `3d-dark-shadow` - Üç boyutlu görüntüleme öğeleri için koyu renkli gölge.
    * `3d-face` - Üç boyutlu görüntüleme öğeleri ve diyalog için yüz rengi kutu arka planları.
    * `3d-highlight` - Üç boyutlu görüntüleme öğeleri için vurgulama rengi.
    * `3d-light` - Üç boyutlu görüntüleme elemanları için açık renk.
    * `3d-shadow` - Üç boyutlu görüntüleme öğeleri için gölge rengi.
    * `active-border` - Etkin pencere kenarı.
    * `active-caption` - Active window title bar. Specifies the left side color in the color gradient of an active window's title bar if the gradient effect is enabled.
    * `active-caption-gradient` - Aktif pencerenin başlık barının gradient renginin içindeki sağ taraf rengi.
    * `app-workspace` Çoklu dosya arayürüz (MDI) uygulamlarının arkaplan rengi.
    * `button-text` - Push butonlarındaki yazı.
    * `caption-text` - Başlığın içindeki yazı, boyut kutusu ve kaydırma çubuğu ok kutusu.
    * `desktop` - Masaüstü arkaplan rengi.
    * `disabled-text` - Gri (devre dışı) metin.
    * `highlight` - Bir kontrolde seçilen öğe(ler).
    * `highlight-text` - Bir kontrol içindeki seçilen öğe(ler)'nin yazısı.
    * `hotlight` - Bir hot-tracked öğe veya hyperlink için renk.
    * `inactive-border` - Aktif olmayan pencere kenarı.
    * `inactive-caption` - Inactive window caption. Specifies the left side color in the color gradient of an inactive window's title bar if the gradient effect is enabled.
    * `inactive-caption-gradient` - Aktif olmayan pencerenin başlık barının gradient renginin içindeki sağ taraf rengi.
    * `inactive-caption-text` - Aktif olmayan bir altyazıdaki metin rengi.
    * `info-background` - Araç ipucu denetimleri için arka plan rengi.
    * `info-text` - Araç ipucu denetimleri için yazı rengi.
    * `menu` - Menü arkaplanı.
    * `menu-highlight` - Menü öğelerini vurgulamak için kullanılan renk düz bir menü olarak görünür.
    * `menubar` - Menüler düz olarak göründüğünde menü çubuğunun arkaplan rengi.
    * `menu-text` - Menüdeki yazılar.
    * `scrollbar` - Kaydırma çubuğu gri alanı.
    * `window` - Pencere arkaplanı.
    * `window-frame` - Pencere çerçevesi.
    * `window-text` - Pencerelerdeki yazılar.
  * On **macOS**
    * `alternate-selected-control-text` - The text on a selected surface in a list or table. _deprecated_
    * `control-background` - The background of a large interface element, such as a browser or table.
    * `control` - The surface of a control.
    * `control-text` -The text of a control that isn’t disabled.
    * `disabled-control-text` - The text of a control that’s disabled.
    * `find-highlight` - The color of a find indicator.
    * `grid` - The gridlines of an interface element such as a table.
    * `header-text` - The text of a header cell in a table.
    * `highlight` - The virtual light source onscreen.
    * `keyboard-focus-indicator` - The ring that appears around the currently focused control when using the keyboard for interface navigation.
    * `label` - The text of a label containing primary content.
    * `link` - A link to other content.
    * `placeholder-text` -  A placeholder string in a control or text view.
    * `quaternary-label` - The text of a label of lesser importance than a tertiary label such as watermark text.
    * `scrubber-textured-background` - The background of a scrubber in the Touch Bar.
    * `secondary-label` - The text of a label of lesser importance than a normal label such as a label used to represent a subheading or additional information.
    * `selected-content-background` - The background for selected content in a key window or view.
    * `selected-control` - The surface of a selected control.
    * `selected-control-text` - The text of a selected control.
    * `selected-menu-item-text` - The text of a selected menu.
    * `selected-text-background` - The background of selected text.
    * `selected-text` - Selected text.
    * `separator` - A separator between different sections of content.
    * `shadow` - The virtual shadow cast by a raised object onscreen.
    * `tertiary-label` - The text of a label of lesser importance than a secondary label such as a label used to represent disabled text.
    * `text-background` - Text background.
    * `text` -  The text in a document.
    * `under-page-background` -  The background behind a document's content.
    * `unemphasized-selected-content-background` - The selected content in a non-key window or view.
    * `unemphasized-selected-text-background` - A background for selected text in a non-key window or view.
    * `unemphasized-selected-text` - Selected text in a non-key window or view.
    * `window-background` - The background of a window.
    * `window-frame-text` - The text in the window's titlebar area.

RGB onaltılık form (`#ABCDEF`) içindeki sistem renk ayarlarını `String` olarak döndürür. See the [Windows docs](https://msdn.microsoft.com/en-us/library/windows/desktop/ms724371(v=vs.85).aspx) and the [MacOS docs](https://developer.apple.com/design/human-interface-guidelines/macos/visual-design/color#dynamic-system-colors) for more details.

The following colors are only available on macOS 10.14: `find-highlight`, `selected-content-background`, `separator`, `unemphasized-selected-content-background`, `unemphasized-selected-text-background`, and `unemphasized-selected-text`.

### `systemPreferences.getSystemColor(color)` _macOS_

* `color` String - One of the following values:
  * `blue`
  * `brown`
  * `gray`
  * `green`
  * `orange`
  * `pink`
  * `purple`
  * `red`
  * `yellow`

Returns `String` - The standard system color formatted as `#RRGGBBAA`.

Returns one of several standard system colors that automatically adapt to vibrancy and changes in accessibility settings like 'Increase contrast' and 'Reduce transparency'. See [Apple Documentation](https://developer.apple.com/design/human-interface-guidelines/macos/visual-design/color#system-colors) for  more details.

### `systemPreferences.isInvertedColorScheme()` _Windows_ _Deprecated_

Returns `Boolean` - `true` if an inverted color scheme (a high contrast color scheme with light text and dark backgrounds) is active, `false` otherwise.

**Deprecated:** Should use the new [`nativeTheme.shouldUseInvertedColorScheme`](native-theme.md#nativethemeshoulduseinvertedcolorscheme-macos-windows-readonly) API.

### `systemPreferences.isHighContrastColorScheme()` _macOS_ _Windows_ _Deprecated_

Returns `Boolean` - `true` if a high contrast theme is active, `false` otherwise.

**Depreacted:** Should use the new [`nativeTheme.shouldUseHighContrastColors`](native-theme.md#nativethemeshouldusehighcontrastcolors-macos-windows-readonly) API.

### `systemPreferences.getEffectiveAppearance()` _macOS_

Returns `String` - Can be `dark`, `light` or `unknown`.

Gets the macOS appearance setting that is currently applied to your application, maps to [NSApplication.effectiveAppearance](https://developer.apple.com/documentation/appkit/nsapplication/2967171-effectiveappearance?language=objc)

**[Kullanımdan kaldırıldı](modernization/property-updates.md)**

### `systemPreferences.getAppLevelAppearance()` _macOS_ _Deprecated_

Returns `String` | `null` - Can be `dark`, `light` or `unknown`.

Gets the macOS appearance setting that you have declared you want for your application, maps to [NSApplication.appearance](https://developer.apple.com/documentation/appkit/nsapplication/2967170-appearance?language=objc). You can use the `setAppLevelAppearance` API to set this value.

**[Kullanımdan kaldırıldı](modernization/property-updates.md)**

### `systemPreferences.setAppLevelAppearance(appearance)` _macOS_ _Deprecated_

* `appearance` String | null - Can be `dark` or `light`

Sets the appearance setting for your application, this should override the system default and override the value of `getEffectiveAppearance`.

**[Kullanımdan kaldırıldı](modernization/property-updates.md)**

### `systemPreferences.canPromptTouchID()` _macOS_

Returns `Boolean` - whether or not this device has the ability to use Touch ID.

**NOTE:** This API will return `false` on macOS systems older than Sierra 10.12.2.

### `systemPreferences.promptTouchID(reason)` _macOS_

* `reason` String - The reason you are asking for Touch ID authentication

Returns `Promise<void>` - resolves if the user has successfully authenticated with Touch ID.

```javascript
const { systemPreferences } = require('electron')

systemPreferences.promptTouchID('To get consent for a Security-Gated Thing').then(success => {
  console.log('You have successfully authenticated with Touch ID!')
}).catch(err => {
  console.log(err)
})
```

This API itself will not protect your user data; rather, it is a mechanism to allow you to do so. Native apps will need to set [Access Control Constants](https://developer.apple.com/documentation/security/secaccesscontrolcreateflags?language=objc) like [`kSecAccessControlUserPresence`](https://developer.apple.com/documentation/security/secaccesscontrolcreateflags/ksecaccesscontroluserpresence?language=objc) on the their keychain entry so that reading it would auto-prompt for Touch ID biometric consent. This could be done with [`node-keytar`](https://github.com/atom/node-keytar), such that one would store an encryption key with `node-keytar` and only fetch it if `promptTouchID()` resolves.

**NOTE:** This API will return a rejected Promise on macOS systems older than Sierra 10.12.2.

### `systemPreferences.isTrustedAccessibilityClient(prompt)` _macOS_

* `prompt` Boolean - whether or not the user will be informed via prompt if the current process is untrusted.

Returns `Boolean` - `true` if the current process is a trusted accessibility client and `false` if it is not.

### `systemPreferences.getMediaAccessStatus(mediaType)` _macOS_

* `mediaType` String - Can be `microphone`, `camera` or `screen`.

Returns `String` - Can be `not-determined`, `granted`, `denied`, `restricted` or `unknown`.

This user consent was not required on macOS 10.13 High Sierra or lower so this method will always return `granted`. macOS 10.14 Mojave or higher requires consent for `microphone` and `camera` access. macOS 10.15 Catalina or higher requires consent for `screen` access.

### `systemPreferences.askForMediaAccess(mediaType)` _macOS_

* `mediaType` String - the type of media being requested; can be `microphone`, `camera`.

Returns `Promise<Boolean>` - A promise that resolves with `true` if consent was granted and `false` if it was denied. If an invalid `mediaType` is passed, the promise will be rejected. If an access request was denied and later is changed through the System Preferences pane, a restart of the app will be required for the new permissions to take effect. If access has already been requested and denied, it _must_ be changed through the preference pane; an alert will not pop up and the promise will resolve with the existing access status.

**Important:** In order to properly leverage this API, you [must set](https://developer.apple.com/documentation/avfoundation/cameras_and_media_capture/requesting_authorization_for_media_capture_on_macos?language=objc) the `NSMicrophoneUsageDescription` and `NSCameraUsageDescription` strings in your app's `Info.plist` file. The values for these keys will be used to populate the permission dialogs so that the user will be properly informed as to the purpose of the permission request. See [Electron Application Distribution](https://electronjs.org/docs/tutorial/application-distribution#macos) for more information about how to set these in the context of Electron.

This user consent was not required until macOS 10.14 Mojave, so this method will always return `true` if your system is running 10.13 High Sierra or lower.

### `systemPreferences.getAnimationSettings()`

`Object` 'i geri getirir:

* `shouldRenderRichAnimation` Boolean - Returns true if rich animations should be rendered. Looks at session type (e.g. remote desktop) and accessibility settings to give guidance for heavy animations.
* `scrollAnimationsEnabledBySystem` Boolean - Determines on a per-platform basis whether scroll animations (e.g. produced by home/end key) should be enabled.
* `prefersReducedMotion` Boolean - Determines whether the user desires reduced motion based on platform APIs.

Returns an object with system animation settings.

## Özellikler

### `systemPreferences.appLevelAppearance` _macOS_

A `String` property that can be `dark`, `light` or `unknown`. It determines the macOS appearance setting for your application. This maps to values in: [NSApplication.appearance](https://developer.apple.com/documentation/appkit/nsapplication/2967170-appearance?language=objc). Setting this will override the system default as well as the value of `getEffectiveAppearance`.

Possible values that can be set are `dark` and `light`, and possible return values are `dark`, `light`, and `unknown`.

This property is only available on macOS 10.14 Mojave or newer.

### `systemPreferences.effectiveAppearance` _macOS_ _Readonly_

A `String` property that can be `dark`, `light` or `unknown`.

Returns the macOS appearance setting that is currently applied to your application, maps to [NSApplication.effectiveAppearance](https://developer.apple.com/documentation/appkit/nsapplication/2967171-effectiveappearance?language=objc)
