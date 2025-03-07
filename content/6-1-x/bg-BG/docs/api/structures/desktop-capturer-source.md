# Обект DesktopCapturerSource

* `id` String - Идентификатор на прозорец или екран, който може да бъде използван като `chromeMediaSourceId` константа при извикване на [`navigator.webkitGetUserMedia`]. Форматът на идентификатора ще бъде `window:XX` или `screen:XX`, където `ХХ` е случайно генерирано число.
* `name` String - Източник на екрана ще бъде наименуван или като `Entire Screen` или `Screen <index>`, докато името на източник на прозорец ще съвпада със заглавието на прозореца.
* `thumbnail` [NativeImage](../native-image.md) - Миниатюрно изображение. **Note:** Няма гаранция, че размера на миниатюрното изображение ще е с размер `thumbnailSize` описан в `options` и изпратен от `desktopCapturer.getSources`. Действителният размер зависи от мащаба на екрана или прозорец.
* `display_id` String - Уникален идентификатор, отговарящ на `id`-то на съответстващия му [Display](display.md), върнат от [Screen API](../screen.md)-то. На някои платформи е еквивалентно на `XX` частта от полето `id` по-горе, но на други се различава. Ще бъде празен низ, ако не е наличен.
* `appIcon` [NativeImage](../native-image.md) - An icon image of the application that owns the window or null if the source has a type screen. The size of the icon is not known in advance and depends on what the the application provides.
