<h1 align="center">Sake</h1>

<p align="center">
  Self-host your reading stack with a clean web library, KOReader sync, and provider-powered book imports.
</p>

![Sake library page](https://raw.githubusercontent.com/Sudashiii/Sake/blob/master/docs/img/webapp/library.png)

## Why Sake?

If you already use KOReader, Sake gives you a much nicer home base around it. You get a self-hosted web library, seamless progress sync, note-aware reading updates, easy book delivery to devices, and a built-in plugin update flow without losing the KOReader setup you already like.

## Features

- Personal library management with covers, metadata, shelves, ratings, and reading state
- Comprehensive reading stats for your library and reading habits
- Rule-based shelves to automatically organize your collection
- Directly download books from multiple provider-based search sources into your collection
- KOReader device pairing and sync from the same self-hosted stack
- Seamless background reading progress sync across devices, including notes and sidecar metadata
- Export an existing KOReader device library back into the web app
- Built-in KOReader plugin updater and release delivery
- Flexible deployment with Docker images or local Bun development
- libSQL plus S3-compatible storage support for managed or fully self-hosted setups
- Progressive Web App for easier access
- OPDS and WebDav Support

OPDS and WebDAV endpoints use HTTP Basic authentication. By default, your normal account password works there. In Settings -> Account, you can optionally set a separate Basic-auth password for those routes while keeping your normal account password valid too.

## First boot

If the database is empty, Sake exposes the normal bootstrap flow in the UI so you can create the first account there. You do not need to predefine a user in the environment.

## KOReader plugin

The KOReader plugin lives in `koreaderPlugins/sake.koplugin`.

Basic setup flow:

1. Install the plugin in KOReader like any other plugin.
2. Open `Settings -> More tools -> Sake`.
3. Open `Setup`.
4. Set the public URL of your Sake web app in `Server URL`.
5. Optionally rename the device in `Device Name`.
6. Choose `Pair Device` and log in with the same username and password you use in the web app.
7. Use the actions in `Sync`, `Library Import/Export`, and `Maintenance` as needed.

The login step exchanges your password for a device API key and clears the password from the device afterward.

You can also export ebooks from the devices home folder back to the web app, including sidecar data such as progress and notes. Great if you have a pre-existing library on your device!

KOReader plugin releases are tracked in the database and the artifacts are served through S3-compatible storage. If the KOReader plugin is updated and you start the app, the new version will be uploaded to S3 so you can use the updater plugin to easily update the core plugin without needing to manually move the files to your KOReader device.

### Concrete Usage
- The Plugin can be found under "Settings" --> "More tools" --> "Sake"
- `Setup` contains `Server URL`, `Device Name`, and `Pair Device` / `Refresh Device Key`
- `Sync` contains `Download New Books`, `Pull Progress From Other Devices`, and `Upload Current Book Progress`
- Books are downloaded when pressing `Download New Books` or when setting the device to sleep
- Progress is automatically uploaded when putting the device to sleep while a book is still open
- If you use multiple devices, use `Pull Progress From Other Devices` to fetch newer progress from another reader
- `Library Import/Export -> Import or Export Existing Library` uploads books already on the device, along with sidecar data such as progress and notes
- Library import/export can take a while, and the device may be unusable until the process finishes
- `Maintenance -> Check for Plugin Updates` checks for new plugin releases
- `Maintenance -> Advanced -> Remote Log Shipping` toggles shipping KOReader logs back to Sake
- Before using the plugin you need to set the server URL and pair the device. Your password is removed from the device after pairing succeeds.
- You can optionally change the auto-generated device name. The device name shows up in the web app for device-specific downloads and API keys.

## Search providers and downloads

Search is provider-based and routed through `POST /api/search`.

- `anna`, `openlibrary`, and `gutenberg` work as normal providers once enabled in `ACTIVATED_PROVIDERS`
- `zlibrary` also requires you to connect your Z-Library session in `Settings -> Logins`

To enable Z-Library support, add it to `ACTIVATED_PROVIDERS`:

```env
ACTIVATED_PROVIDERS=zlib,anna,openlib,gutenberg
```

In the app UI, open `Settings -> Logins`, then use `Connect Z-Library` and either:

- log in with your email and password, or
- copy `remix_userid` and `remix_userkey` from your Z-Library cookies

The app uses those session values for authenticated Z-Library requests.