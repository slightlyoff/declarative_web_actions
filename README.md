# Declarative Web Actions Explained

> March 8th, 2019<br>
> Last Update: March 8th, 2019
>
> Alex Russell <code>&lt;slightlyoff@google.com&gt;</code><br>
> Frances Berriman <code>&lt;frances@fberriman.com&gt;</code><br>

## What’s all this then?

Declarative Web Actions (DWAs) are an extension to [Web App Manifests](https://developer.mozilla.org/en-US/docs/Web/Manifest) that support in-app actions (e.g., menu and keyboard-driven commands) as well as external assistant-driven interactions with services.

DWAs build on the HTTP-centric [pattern we contributed to the Web Share Target](https://wicg.github.io/web-share-target/level-2/), adding a document-side set of events and interfaces to handle actions in-app.

These extensions seek to enable developers to surface more understanding about what's possible within applications to runtimes and crawlers. Deeper integration with assistive and a11y technology may result from broad DWA deployment.

### Goals

Declarative Web Actions should enable:

 * The ability to represent in-document actions that are drive both by touch interactions as well as keyboard shortcuts (e.g. `ctl-b` for bold)
 * Assistant-driven hotword and action integration
 * A way to surface "long-press" and jumplist menu options
 * Runtime enabling/disabling of actions based on focus
 * Through the same basic interface, a way to allow a11y tools to drive applications

### Non-goals

It is not a goal to:

 * Actions for non-PWA content
 * Faithfully absract over all existing conversational APIs
 * Manage application authentication or authorization
 * Handle localization in-format; we expect developers to serve alternative manifest files per locale or language

## Key scenarios


### PWA Menus &amp; Keyboard Accelerators

TODO

```json
TODO
```

### External Assistants

TODO

```json
TODO
```

### Long-Press/Jumplist Menus

TODO

```js
TODO
```

## Detailed Design Discussion

TODO

## Open Design Questions

  * There needs to be API to enable/disable DWAs...what about maniuplating the hierarchy? How dynamic should that be in-document?
  * The evented side of this design is too complex because we don't have an `onbeforenavigate` event yet. Maybe we should just add one here?
  * Should params be able to be specified as [Schema.org types](https://schema.org/docs/full.html)?
  * Should the `actions` collection be a flat list? Windows uses a [`GroupName`](https://docs.microsoft.com/en-us/uwp/api/windows.ui.startscreen.jumplistitem) concept to avoid needing to manually re-create a hierarchy. Perhaps this is better?
  * It's unclear how to model ongoing conversational call/response in this API across in-document and across-document models (SPA vs. MPA). At the SW/HTTP level, it's pretty straightforward to have assistant systems send along some sort of an "I expect JSON in response" thing, but in the context of documents the ongoing conversation flow isn't as clear.
  * Keyboard shortcut handling is hard disambiguate. Do we need to support `key` and `location` in addition to `code`? Have shied away from creating a new micro-syntax that needs it's own parsing in lieu of keyboard event constructor options, but that may not be enough?
  * How do we handle reserved or overwritten kbd shortcuts from the OS?
  * Should there be a system for registering [global keyboard handlers](https://developer.chrome.com/apps/commands)?
  * The use of HTTP status codes for success/response requires us to double-up the surface area to provide something on the evented side of things.

## Considered alternatives

### Purely Programmatic APIs

TODO

## References & acknowledgements

References:

  * [Web Share Target Level 2](https://wicg.github.io/web-share/level-2/)
  * [WHATWG DOM](https://dom.spec.whatwg.org/)
  * [UI Events](https://www.w3.org/TR/uievents/)
  * [Android Shortcuts](https://developer.android.com/guide/topics/ui/shortcuts) 
  * [Windows Jump Lists](https://docs.microsoft.com/en-us/uwp/api/windows.ui.startscreen.jumplist)
  * [Windows UWP Keyboard Accelerators](https://docs.microsoft.com/en-us/windows/uwp/design/input/keyboard-accelerators)
  * [`chrome.commands`](https://developer.chrome.com/apps/commands)
  * [Apple kbd interaction gidelines](https://developer.apple.com/design/human-interface-guidelines/macos/user-interaction/keyboard/)
  * [Apple Keyboard and Menu APIs](https://developer.apple.com/documentation/uikit/keyboard_and_menus?language=objc)
  * [Google Assistant Actions](https://developers.google.com/actions/extending-the-assistant)
  * [Schema.org types](https://schema.org/docs/full.html)



We'd like to acknowledge the contributions of:

  - ...
