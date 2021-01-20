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
