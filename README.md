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

Desktop PWAs are currently missing a way to provide system-integrated menus and keyboard accelerators. To improve this situation, we propose extensions to the [Web App Manifest](https://developers.google.com/web/fundamentals/web-app-manifest/) that all PWAs much include. As a refresher, web pages that wish to be installed must like to a manifest in the `<head>` of their documents, like this:

```html
<link rel="manifest" href="/manifest.json">
```

Manifests must include a few properties to be treated as installable, e.g.:

```json5
{
  "name": "Frobulator Deluxe",
  "short_name": "Frobulator",
  "icons": [
     {
      "src": "/app/images/icons-512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ],
  "start_url": "/app/?source=pwa",
  "background_color": "#3367D6",
  "display": "standalone",
  "scope": "/app/",
  "theme_color": "#3367D6",
  ...
}
```

To implement menuing, we add an `actions` section:

```json5
{
  "name": "Frobulator Deluxe",
  "short_name": "Frobulator",

  ...

  "actions": [
    {
      "name": "Start",
      "action": "/app/start",
    }
  ],
}
```

Each item, by default, invokes a navigation (HTTP GET) to the listed location in the `action` field. This may not be desireable, and so an event is dispatched within open documents in the PWA whose default behavior can be prevented:

```js
window.addEventListener("action", (e) => {
  // Handle the interaction locally, do not navigate:
  e.preventDefault();
  // ...
  console.log(e.name);
  console.log(e.action);
  console.log(e.formData); // empty
});
```

Applications frequently want to map actions to user actions in a more generic way. Keyboard accelerators and menu options are mutliple routes for accessing the same functions, however they admit some per-platform diversity. Here's a sketch for how we might enable the same action to be accessible in both places.


```json5
{
  "name": "Frobulator Deluxe",
  "short_name": "Frobulator",

  ...

  "actions": [
    {
      "name": "Start",
      "action": "/app/start",
      "method": "GET",

      "keys": {
        "default": { // Shift-Ctrl-S
          "code": "S",
          "ctrlKey": true,
          "shiftKey": true, 
        },
        "macos": {   // Command-S on MacOS
          "code": "S",
          "metaKey": true,
        }
      }
    }
  ],
}
```

> TODO(slightlyoff): describe HTTP response code semantics for success failure

> TODO(slightlyoff): outline a DOM API for enabling/disabling items as well as an `id` concept. 

> OPEN ISSUE: the reification of these canned menus into DOM `<menu>`s seems, in some ways, obvious. Should they simply be exposed as DOM hanging off of `<head>` or `document.documentElement` in some way?

> OPEN ISSUE: global keyboard handling (OS-registered shortcuts that cause an app to be launched even if it's not already running) are not described here as their priority is unclear. Need feedback to understand the importance.

### External Assistants

Most of today's voice-driven assistants are, conceptually, form-filler-outers. They are invoked by hotwords and lean on canned sentence structures to fill in the gaps when the required variables aren't provided.

Supporting this style of operation as well as more complex, free-form interaction is a goal of this proposal. To do this, we extend the HTTP-equivalent actions (in the style of Web Share Target) to include `POST` operations and build in type-awareness based on [Schema.org's vocabulary of extended type hints](https://schema.org/docs/full.html). There are many open questions as to how these will interoperate.

```json5
// A more complex action with a keyboard shortcut, a jumplist, and
// arguments
{
  "name": "Book A Local Frobulator",
  "short_name": "Book",
  "hotwords": [ "book", "reserve"],

  "action": "/app/book",
  "method": "POST",
  "enctype": "multipart/form-data",
  "params": {
    "location": {
      // We extend the params mapping to allow descriptive phrases for
      // each param. This is necessary for assistant-driven form
      // filling.
      "field": "location",
      "expect": [ "near me", "around me" ],
    },
    "time": {
      "field": "time",
      // TODO: does this make sense?
      "schema": "https://schema.org/Time",
    },
    "time_separator": {
      // Used to construct voice query, not submitted (no `field`)
      "expect": [ "at", "around" ]
    },
  },
  "expect": [ 
    "${params.location} ${time_separator} ${params.time}",
    // ...
  ],

},
```

### Long-Press/Jumplist/Context Menus

We can build on the previous example to outline how some actions might also be surfaced outsie the application context, e.g. in OS-level "jumplists" or "long-press" menus that invoke the service even if it isn't currently running.

> TODO(slightlyoff): reconcile this sort of invocation with assistant action filling. E.g., if an assistant would normally `POST` a full form's worth of data, but a jumplist or hotkey invocation won't, how do we represent these alternative paths? Separate actions with the same name? An `optional` flag on `params`?

```json5
// A more complex action with a keyboard shortcut, a jumplist, and
// arguments
{
  "name": "Book A Local Frobulator",
  "short_name": "Book",

  // TODO: does this action `GET` with empty query params when invoked by a
  // kbd shortcut or hotlist/shortcut system?
  "action": "/app/book",
  "method": "POST",

  // ...

  "context-menu": true,
  // Surface in jumplist/long-press? (default `false`)
  "shortcut": true,

  // Icons for a jumplist or long-press UI
  "icons": [
    {
      "src": "/app/images/book-192.png",
      "type": "image/png",
      "sizes": "192x192"
    },
  ],

  // ...
},
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
