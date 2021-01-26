# Subapps Explainer

Authors:
* Ivan Šandrk &lt;<isandrk@chromium.org>&gt;
* Philipp Weiß‎ &lt;<phweiss@chromium.org>&gt;

## Introduction

In some circumstances it may be required for a single PWA to create multiple launch icons with distinct names. This allows a single software package to represent different functionality to the user but still be a single PWA install.

## Example use cases

* A virtual desktop PWA can allow a user to run applications on a remote server that manifest local client windows. For example, a user might be running a remote word processing app, a remote terminal application and a remote web browser. All of the remote apps are manifested by a single virtual desktop PWA, but each should be in its own standalone window, with a distinct name and launch icon. The distinct icon should appear on all surfaces that list installed apps (application menu / launch menu / desktop / home screen) and that list running apps, if the app is running (task bar / shelf / dock / overview menu).
* Office productivity suites offer a wide range of different functionality including an email client, word processor, spreadsheet editor and more. Users of office suites typically use more than one major feature and it would be inconvenient to make the user install each as a separate PWA. With a multi-icon PWA, the user can install once and then access launch icons for distinct functionality.

## Goals

Enable PWA developers to programmatically add application icons to their PWA that link into specific application features. This is to provide the user experience of using separate apps that seamlessly integrate into the work environment, even though they might be provided by a single PWA. For example, the user should see distinct Word, Excel, etc. icons, and not a single VDI Provider icon with a popup menu with “Launch Word”, “Launch Excel”, etc. actions.

Programmatic management of icons is required to handle the use case of PWAs that add or remove functionality over time, for example, an enterprise Administrator adding newly licensed software through VDI Provider or similar virtual desktop application as described in example use cases.

## Non-goals

It is not a goal to support features offered to an installed PWA for ‘supplemental icons’ including:

* badging (except possibly a forced badge of the PWA’s original icon for security reasons)
* display mode
* independent scope
* browser handling of launcher icon clicks
* tray/menu/notification icons
* context menu shortcuts (action/jumplist shortcuts)

## Considered approaches

In the following subsections we take a closer look at the following possible approaches:

* Manifest URL - shortcuts to sub-parts of the main app are implemented as sub-apps (each having its own manifest)
* JSON object - similar to Manifest URL
* Large manifest - shortcuts are implemented statically in the manifest of the main app

Weighing the pros and cons of each we pick the _Manifest URL_ as the most viable approach.

### Manifest URL

The client app will add sub-apps via something like the following, which would in essence _install_ the sub-app almost the same way as a regular app:
```javascript
spreadsheet_url = "https://vdi.app/spreadsheet/manifest.json";

await navigator.addRelatedApplication(spreadsheet_url);

// /spreadsheet/manifest.json
{
  "name": "Spreadsheet",
  "icons": [{
    "src": "/images/icons/spreadsheet.png",
  }],
  "start_url": "/spreadsheet/",
  "display": "standalone"
}
```

This API call would be restricted to same-origin manifests as the document calling it.

**Pros:**
* Most congruent with the architecture, least amount of dev work needed
* Can reuse existing specs without needing to change them to support these new features
* Lightweight on the security review side
* Don't need to duplicate functionality of manifest parts into sub-app manifest keys
* Future APIs that end up in the manifest will show up in the sub-app manifest for free
* Very dynamic and gives a lot of control to app developers
   * Eg. app devs can change File handler associations immediately (Excel gets installed as a sub-app, and the File handler is associated immediately with it)
* API does not need to distinguish between updating and adding as the manifest url (currently start_url) will be the unique identifier

**Cons:**
* Clients of the API need to create and host the manifest file, but if needed this can be circumvented by clever usage of the service worker which can generate and serve the manifest on the fly

### JSON object

Very similar to the Manifest URL approach, except the manifest is directly fed to the API call (instead of indirectly through a URL).  The client app will add sub-apps via something like the following:
```javascript
spreadsheet = {
  "name": "Spreadsheet",
  "icons": [{
    "src": "/images/icons/spreadsheet.png",
  }],
  "start_url": "/spreadsheet/",
  "display": "standalone"
};

await navigator.addRelatedApplication(spreadsheet);
```

**Pros (compared to the Manifest URL approach):**
* Clients of API don’t need to serve the manifest

**Cons (compared to the Manifest URL approach):**
* The already existing code expects the manifest to be given via a URL, adapting it to this approach would incur significant development costs
* The requirement of the developer providing an ID, or having the API shape distinguish between updating and adding

### Large manifest

Do everything in the manifest. The sub-apps/shortcuts for the main app would be specified in its manifest statically. The manifest of the app would look something like the following:
```javascript
{
  "name": "Office VDI app",
  "start_url": "https://vdi.app/",
  "shortcuts": [{
    "name": "Word processor",
    "url": "/word-processor",
    "icon": "/icons/word-processor.svg",
    // Since jumplist (context menu) shortcuts already [exist](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/main/Shortcuts/explainer.md).
    "location": "launcher",
  }, {
    "name": "Spreadsheet",
    "url": "/spreadsheet",
    "icon": "/icons/spreadsheet.svg",
    "location": "launcher",
  }]
}
```

This would install the main app with its app icon, and also two shortcut icons (word/spreadsheet) for the sub-apps.

**Pros:**
* Partial manifest spec for how shortcuts should be handled, but this is incomplete

**Cons:**
* Requires adapting the manifest spec format to support sub-app keys
  * Spec changes + security review
  * Code changes, bake time
* Single install/uninstall events for the main app that won't happen often -> requires the APIs in use be more dynamic to support extra functionality
  * main-app v1 is launched
  * At some point, main-app v2 comes out that adds a new sub-app
  * Will need to wait ~1 day for an update to come to install the app
* Might put a lot of burden on clients of API as they might want to customize app installs (different groups of users with different sets of sub-apps) - a lot of different manifests to serve
* Install & uninstall of subapps would require a manifest update, which is throttled currently & would have to be re-designed
* Updates of subapps would follow suit - require a manifest update of parent app, which can only happen once a day
  * Would also require a unique id per sub app - how will the browser know which one is new and which one is updating (if we move away from start_url)

### Security & Privacy considerations

* To prevent spoofing, both the sub-app name/icon would be badged together with the name/icon of the original PWA (ie. “Text Processor (VDI Provider)”)
* `add` API call triggers a user agent permission prompt asking the user to confirm the action, as spam and spoofing prevention. The prompt would prominently include the shortcut name and icon.

### Open Questions

* Badging - we would want to security-badge the new shortcut app icon, but ideally this could work together with the already existing Badging API - how to go about doing this?
  * another problem is how does the Badging API choose the right icon to badge? currently it just goes for the app itself, but in the context of a sub-app this might do the wrong thing and add the badge onto the main app icon instead of the sub-app
  * We could possibly choose the right app by looking at the longest-scoped-installed-app for the given document the API is called on.

### Future Work

* DLC (declarative link capturing) & SWLE (service worker launch event) - both should work out of the box once available

### References

[sw-launch Event Explainer](https://wicg.github.io/sw-launch/explainer.html)
* currently web apps have no control over whether launches will happen in a new window or an existing one
* this feature allows Service Workers to control which window/tab they will open with

[Declarative Link Capturing (DLC)](https://github.com/WICG/sw-launch/blob/master/declarative_link_capturing.md)
* An alternative to sw-launch that is less powerful, but declarative, and has the option of expanding into the full launch event later on (“lightweight” SWLE)

### Acknowledgements

* Alex Russell‎
* Chase Phillips‎
* Daniel Murphy
* Dominic Farolino
* Joshua Bell
* Matt Giuca
