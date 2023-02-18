# Scrollmus
Framework-agnostic scrollspy script, fork of [Gumshoe by cferdinandi](https://github.com/cferdinandi/gumshoe). The latest compiled release can be found in the `dist` directory.

What's different?

+ Added `useLast` toggle [GS#128](https://github.com/cferdinandi/gumshoe/issues/128)
+ Merged [GS#111](https://github.com/cferdinandi/gumshoe/issues/111) & [GS#117](https://github.com/cferdinandi/gumshoe/issues/117)
+ Removed polyfill support (Internet Explorer)

## Browser Support
Scrollmus supports all modern browsers; this does not include Internet Explorer.

| Firefox | Chrome | Edge  | Safari | IE    |
| :---:   | :---:  | :---: | :---:  | :---: |
| 35+     | 41+    | 15+   | 9+     | --    |

## Include

### Static
Download the latest compiled release from the `dist` directory.

```html
<script defer src="/path/to/scrollmus.min.js"></script>
```

### CDN
Scrollmus can be loaded using a CDN of choice. It is recommended to use a specific version range to prevent major updates from breaking your site. Scrollmus uses semantic versioning.

Refer to the [jsDelivr NPM CDN feature documentation](https://www.jsdelivr.com/features#npm) for information on version ranges.

#### jsDelivr

```html
<script defer src="https://cdn.jsdelivr.net/npm/scrollmus/dist/scrollmus.min.js"></script>
```

#### unpkg

```html
<script defer src="https://unpkg.com/scrollmus/dist/scrollmus.min.js"></script>
```

### Hugo
Scrollmus can be loaded as a Hugo Module.

Initialize the Hugo project as a Hugo Module. Replace the path with your repository.

```bash
hugo mod init gitlab.com/username/project
```

Add the following `[module]` configuration to the project's `config.toml` file. If the `gitlab.com` import fails for your project, replace it with `github.com`. Scrollmus is distributed on both platforms.

```toml
[module]
  [[module.imports]]
    path = 'gitlab.com/aao-fyi/scrollmus'
  [[module.imports.mounts]]
    source = 'dist/'
    target = 'assets/js/scrollmus/'
```

Load Scrollmus wherever necessary.

```html
{{ $scrollmus := resources.Get "js/scrollmus/scrollmus.min.js" }}
<script defer src="{{ $scrollmus.RelPermalink }}"></script>
```

Optionally use [subresource integrity](https://developer.mozilla.org/docs/web/Security/Subresource_Integrity).

```html
{{ $scrollmus := resources.Get "js/scrollmus/scrollmus.min.js" }}
{{ $scrollmusSRI := $scrollmus | resources.Fingerprint "sha384" }}
<script defer src="{{ $scrollmusSRI.RelPermalink }}" integrity="{{ $scrollmusSRI.Data.Integrity }}"></script>
```

### NPM
The wombats of the group can use NPM.

```bash
npm install scrollmus
```

## Usage
The only thing Scrollmus needs to work is a list of anchor links. They can be ordered or unordered, inline or unstyled, or even nested.

```html
<ul id="my-awesome-nav">
	<li><a href="#eenie">Eenie</a></li>
	<li><a href="#meenie">Meenie</a></li>
	<li><a href="#miney">Miney</a></li>
	<li><a href="#mo">Mo</a></li>
</ul>
```

In the footer of your page, after the [include](#include), initialize Scrollmus by passing in a selector for the navigation links that should be detected as the user scrolls.

```html
<script>
	var spy = new Scrollmus('#my-awesome-nav a');
</script>
```

## Styling
Scrollmus adds the `.active` class to the list item (`<li></li>`) and content for the active link, but does not include any styling.

Add styles to your CSS as desired. And that's it, you're done. Nice work!

```css
#my-awesome-nav li.active a {
	font-weight: bold;
}
```

*__Note:__ you can customize the class names with [user options](#options-and-settings).*

## API
Scrollmus includes smart defaults and works right out of the box. But if you want to customize things, it also has a robust API that provides multiple ways for you to adjust the default options and settings.

### Options and Settings
You can pass options into Scrollmus when instantiating.

```js
var spy = new Scrollmus('#my-awesome-nav a', {

	// Active classes
	navClass: 'active', // applied to the nav list item
	contentClass: 'active', // applied to the content

	// Nested navigation
	nested: false, // if true, add classes to parents of active link
	nestedClass: 'active', // applied to the parent items

	// Offset & reflow
	offset: 0, // how far from the top of the page to activate a content area
	reflow: false, // if true, listen for reflows

	// Event support
	events: true, // if true, emit custom events

	// End of page
	useLast: true // if true, the last page item will be set as 'active' when scrolled to bottom
});
```

### Custom Events
Scrollmus emits two custom events:

+ `scrollmusActivate` is emitted when a link is activated.
+ `scrollmusDeactivate` is emitted when a link is deactivated.

Both events are emitted on the list item and bubble up. You can listen for them with the `addEventListener()` method. The `event.detail` object includes the `link` and `content` elements, and the `settings` for the current instantiation.

```js
// Listen for activate events
document.addEventListener('scrollmusActivate', function (event) {

	// The list item
	var li = event.target;

	// The link
	var link = event.detail.link;

	// The content
	var content = event.detail.content;

}, false);
```

### Methods
Scrollmus also exposes several public methods.

#### setup()
Setups all of the calculations Scrollmus needs behind-the-scenes. If you dynamically add navigation items to the DOM after Scrollmus is instantiated, you can run this method to update the calculations.

```js
var spy = new Scrollmus('#my-awesome-nav a');
spy.setup();
```

#### detect()
Activate the navigation link that's content is currently in the viewport.

```js
var spy = new Scrollmus('#my-awesome-nav a');
spy.detect();
```

#### destroy()
Destroy the current instance of Scrollmus.

```js
var spy = new Scrollmus('#my-awesome-nav a');
spy.destroy();
```

## Examples

### Nested Navigation
If you have a nested navigation menu with multiple levels, Scrollmus can also apply an `.active` class to the parent list items of the currently active link.

```html
<ul id="my-awesome-nav">
	<li><a href="#eenie">Eenie</a></li>
	<li>
		<a href="#meenie">Meenie</a>
		<ul>
			<li><a href="#hickory">Hickory</a></li>
			<li><a href="#dickory">Dickory</a></li>
			<li><a href="#doc">Doc</a></li>
		</ul>
	</li>
	<li><a href="#miney">Miney</a></li>
	<li><a href="#mo">Mo</a></li>
</ul>
```

Set `nested` to `true` when initializing. This class name can also be customized.

```js
var spy = new Scrollmus('#my-awesome-nav a', {
	nested: true,
	nestedClass: 'active-parent'
});
```

### Catching Reflows
If the content that's linked to by your navigation has different layouts at different viewports, Scrollmus will need to detect these changes and update some calculations behind-the-scenes.

Set `reflow` to `true` to enable this (it's off by default).

```js
var spy = new Scrollmus('#my-awesome-nav a', {
	reflow: true
});
```

### Fixed Headers
If you have a fixed header on your page, you may want to offset when a piece of content is considered "active."

The `offset` user setting accepts either a number, or a function that returns a number. If you need to dynamically calculate dimensions, a function is the preferred method.

Here's an example that automatically calculates a header's height and offsets by that amount.

```js
// Get the header
var header = document.querySelector('#my-header');

// Initialize Scrollmus
var spy = new Scrollmus('#my-awesome-nav a', {
	offset: function () {
		return header.getBoundingClientRect().height;
	}
});
```

### Use Last Item
By default, when the user scrolls to the bottom of the page the last item will be marked active. To prevent this behavior, set `useLast` to false when you call Scrollmus. When `useLast` is false, the item at the top of the page will continue to be marked active.

```js
var spy = new Scrollmus('#my-awesome-nav a', {
	useLast: false
});
```

## Issues
Open new issues in the [GitLab Issue Tracker](https://gitlab.com/aao-fyi/scrollmus/-/issues).

## License
Scrollmus is distributed under the [MIT License](https://gitlab.com/aao-fyi/scrollmus/-/blob/main/LICENSE).
