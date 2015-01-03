# mq4-hover-hover-shim
[![NPM version](https://badge.fury.io/js/mq4-hover-hover-shim.svg)](http://badge.fury.io/js/mq4-hover-hover-shim)
[![Build Status](https://img.shields.io/travis/cvrebert/mq4-hover-hover-shim/master.svg)](https://travis-ci.org/cvrebert/mq4-hover-hover-shim)
[![Dependency Status](https://david-dm.org/cvrebert/mq4-hover-hover-shim.svg)](https://david-dm.org/cvrebert/mq4-hover-hover-shim)
[![devDependency Status](https://david-dm.org/cvrebert/mq4-hover-hover-shim/dev-status.svg)](https://david-dm.org/cvrebert/mq4-hover-hover-shim#info=devDependencies)

A shim for the [Media Queries Level 4 `hover` @media feature](http://drafts.csswg.org/mediaqueries/#hover).

The CSSWG's [Media Queries Level 4 Working Draft](http://drafts.csswg.org/mediaqueries/) defines a [`hover` media feature](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/hover) that can be used in media queries. This can be used to determine whether the user-agent's primary pointing device truly supports hovering (like mice do) (the `hover` value), or emulates hovering (e.g. via a long tap, like most modern touch-based mobile devices) (the `on-demand` value), or does not support hovering at all (like some old mobile devices) (the `none` value). This matters because emulated hovering typically has some ugly quirks, such as [`:hover`](hover-pseudo) being "sticky" (i.e. a hovered element stays in the [`:hover` state](hover-pseudo) even after the user stops interacting with it and until the user hovers over a different element). It is often better to avoid `:hover` styles in browsers where hovering supports is emulated.

However, since it's from a relatively recent Working Draft, the `hover` media feature is not supported in all current modern browsers or in any legacy browsers. So, this library was created to shim support for the feature into browsers that lack native support for it.

NOTE: This shim only adds support for the `hover` value of the `hover` media feature. So you can only tell the difference between "truly supports hovering" (the `hover` value)" and "does not truly support hovering" (the `none` or `on-demand` values).

The shim consists of two parts:
* A [PostCSS](https://github.com/postcss/postcss)-based server-side CSS postprocessor that rewrites
```css
@media (hover: hover) {
    some-selector {
        property: value;
    }
}
```
into
```css
some-prefix some-selector {
    property: value;
}
```
(In normal use-cases, `some-selector` will contain the `:hover` pseudo-class and `some-prefix` will be a specially-named CSS class that will typically be added to the `<html>` element.)
* A client-side JavaScript library that detects whether the user-agent truly supports hovering. If the check returns true, then your code can add the special CSS class to the appropriate element to enable [`:hover`](hover-pseudo) styles; for example:
```js
if (mq4HoverShim.supportsTrueHover()) {
    document.documentElement.className += ' some-special-class';
}
```
Obviously, this requires JavaScript to be enabled in the browser, and would default to disabling `:hover` styles when JavaScript is disabled.

[hover-pseudo]: https://developer.mozilla.org/en-US/docs/Web/CSS/:hover

## Browser compatibility

The following is a summary of the results of testing the library in various browsers. [Try out the Live Testcase](http://jsfiddle.net/cvrhulu/5vszkpmg/).

Legend:
* True positive - Browser supports real hovering, and mq4-hover-hover-shim reports that it supports real hovering
* True negative - Browser does NOT support real hovering, and mq4-hover-hover-shim reports that it does NOT support real hovering
* False negative - Browser supports real hovering, and mq4-hover-hover-shim reports that it does NOT support real hovering
* False positive - Browser does NOT supports real hovering, and mq4-hover-hover-shim reports that it supports real hovering
* ??? - This case has yet to be tested.
* Desktop - has a pointing device that supports true hovering (e.g. mouse, trackball, trackpad, joystick, http://xkcd.com/243/); lacks a touch-based pointing input device
* [Laplet](http://en.wikipedia.org/wiki/Laplet) - has both a pointing device that supports true hovering and a touch-based pointing input device
* Mobile - has a touch-based pointing input device (e.g. touchscreen); lacks a pointing device that supports true hovering

Officially supported:
* Blink (Chrome & recent Opera)
  * Desktop - True positive in Chrome >=41; False negative in Chrome <41 due to [Chromium bug #441613](http://crbug.com/441613)
  * Laplet - ??? (Arguable true negative presumed)
  * Mobile (Android) - True negative
* Firefox
  * Desktop - True positive
  * Laplet - ??? (Arguable true negative presumed)
  * Mobile (Android) - True negative
* Android browser
  * Mobile (Android 4.0/5.0) - True negative
  * Laplet (Android 4.0/5.0) - ??? (Arguable true negative presumed)
* Internet Explorer
  * Desktop
    * 11 - True positive
    * 10 - True positive
    * 9 - True positive
    * 8 - True positive
  * Laplet
    * 11 - Arguable true negative
  * Mobile (Windows Phone 8.1)
    * 11
      * in mobile mode - True negative
      * in desktop mode - True negative
* Safari (WebKit)
  * Desktop (Safari 8 on OS X) - True positive
  * Mobile (iOS 8.1) - True negative

Unofficially supported:
* Presto
  * Desktop (old Opera 12.1) - True positive
  * Mobile (Opera Mini) - ??? (Theoretically: True negative)
  * Mobile (Opera Mobile) - ??? (Theoretically: True negative)
* Internet Explorer Mobile <=10 - ??? (Theoretically: True negative)

## API
### Node.js module; CSS postprocessor
The npm module has the following properties:
* `postprocessor` - CSS postprocessor that transforms the source CSS as described above. A [PostCSS](https://github.com/postcss/postcss) processor object (that was returned from a call to the `postcss()` function). It requires that a `hoverSelectorPrefix` string option be provided; this string will be prepended to all selectors within `@media (hover: hover) {...}` blocks within the source CSS.
* `featureDetector` - Each of this object's properties is a string filepath to a JavaScript file containing the browser-side feature detector in a particular JavaScript module format.
  * `es6` - [ECMAScript 6 module](http://www.2ality.com/2014/09/es6-modules-final.html) format (this is the original from which the other versions are generated)
  * `cjs` - [CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1) module format
  * `umdGlobal` - "enhanced" [UMD](https://github.com/umdjs/umd) module format; exports a `window.mq4HoverShim` global if the JS environment has no module system (e.g. if included directly via `<script>` in current browsers); (generated via Browserify's `standalone` option)

### Browser-side feature detector
When used in a non-AMD non-CommonJS context, the module exports itself as a `mq4HoverShim` property on the global `window` object.
The module exports one public function:
* `supportsTrueHover()`
  * Arguments: none
  * Side-effects: none
  * Return type: `boolean`
  * Returns a `boolean` value indicating if the browser's primary pointer supports true hovering or if the browser at least does not try to quirkily emulate hovering, such that [`:hover`](hover-pseudo) CSS styles are appropriate.
  * In other words, returns `true` if `@media (hover: hover)` would evaluate to `true` were the browser to natively correctly implement Media Queries Level 4; otherwise, returns `false`.
  * If the browser does not natively support the `hover` media feature, but does support touch via some pointing input device, then we define this touch-based pointer to be the "primary pointer". Hence, if said browser has multiple pointing input devices, one supporting touch and another supporting true hovering (e.g. the computer has both a mouse and a touchscreen), this function will return `false`, since the user could use the touch input device at any time and since `:hover` should only be used for progressive enhancement anyway.

## Grunt
Use [grunt-postcss](https://github.com/nDmitry/grunt-postcss) to invoke the mq4-hover-hover-shim CSS postprocessor via [Grunt](http://gruntjs.com/) task.

## Contributing
The project's coding style is laid out in the JSHint, ESLint, and JSCS configurations. Add unit tests when changing the CSS postprocessor. Lint and test your code using [Grunt](http://gruntjs.com/). Manually test any changes to the browser-side portion of the shim.

_Also, please don't edit files in the `dist` subdirectory as they are generated via Grunt. You'll find source code in the `src` subdirectory!_

## Release History
See the [GitHub Releases page](https://github.com/cvrebert/mq4-hover-hover-shim/releases) for detailed changelogs.
* (next release) - `master`
* 2014-12-31 - v0.0.1: Initial release

## License
Copyright (c) 2014-2015 Christopher Rebert. Licensed under the MIT License.
