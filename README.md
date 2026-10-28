# Kodi HDMI CEC Power Controls

A simple Kodi script addon that lets you trigger HDMI CEC power commands via Kodi's JSON-RPC API.

## What this addon does

It exposes script actions that map to Kodi built-ins:

- `on` -> `CECActivateSource` (wake / switch TV input)
- `off` -> `CECStandby` (put TV/CEC chain in standby)

## Install

Installation and updates are managed through the [shinn5112 Kodi repository](https://github.com/shinn5112/shinn5112-kodi-repo). Installing from it means Kodi will notify you of, and install, new releases automatically.

1. Download the repository add-on: [repository.shinn5112-1.0.0.zip](https://raw.githubusercontent.com/shinn5112/shinn5112-kodi-repo/main/repository.shinn5112/repository.shinn5112-1.0.0.zip)
2. In Kodi: **Add-ons -> Install from zip file** and select the downloaded zip. (You may need to enable installs from unknown sources first.)
3. Go to **Add-ons -> Install from repository -> shinn5112 Kodi Repository** and install **HDMI CEC Power Controls**.
4. Ensure HDMI CEC is enabled on your TV and in Kodi.

Requires Kodi 20 (Nexus) or newer.

### Adopt this add-on into the repository

1. In `shinn5112-kodi-repo`, open **Settings -> Secrets and variables -> Actions -> Variables**.
2. Add this line to the `ADDON_SOURCES` variable (whitespace or newline separated, alongside any existing entries):

   ```
   https://github.com/shinn5112/kodi-hdmi-cec-power-controls.git#main
   ```

   If `ADDON_SOURCES` is unset, it defaults to only the Crunchyroll plugin, so include that entry as well when you first set the variable.
3. Add a `KODI_REPO_TOKEN` secret to this repository (see below), then run the **Build Kodi repository** workflow from the Actions tab. It publishes `script.module.hdmi-cec-power-controls/` with the zip, `addons.xml` and its checksum to `main`.

The notify workflow needs a `KODI_REPO_TOKEN` secret in this repository: a personal access token with contents write access to `shinn5112-kodi-repo`. To target a different Kodi repository, set a `KODI_REPO` Actions variable (`owner/name`).

## Release a new version

1. Update `version` and the `<news>` entry in `addon.xml`. Kodi only offers an update when the version increases.
2. Merge to `main`.
3. Publish a GitHub release (tag it with the add-on version, for example `0.2.0`). The **Notify Kodi repository** workflow runs on published releases only, not on pushes or pre-releases, and triggers the **Build Kodi repository** workflow in `shinn5112-kodi-repo`, which publishes the new zip. It can also be started manually from the Actions tab.
4. Kodi picks up the new version the next time it checks for add-on updates.

## JSON-RPC examples

Kodi JSON-RPC endpoint is usually:

- `http://<kodi-host>:8080/jsonrpc`

### Turn TV on

```bash
curl -X POST http://<kodi-host>:8080/jsonrpc \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "Addons.ExecuteAddon",
    "params": {
      "addonid": "script.module.hdmi-cec-power-controls",
      "params": ["on"]
    }
  }'
```

### Turn TV off

```bash
curl -X POST http://<kodi-host>:8080/jsonrpc \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "Addons.ExecuteAddon",
    "params": {
      "addonid": "script.module.hdmi-cec-power-controls",
      "params": ["off"]
    }
  }'
```

## Optional argument formats

The addon accepts these equivalent action formats:

- `"on"`
- `"action=on"`
- `"--action=on"`

## Notes

- CEC behavior is device-dependent. Some TVs ignore wake/standby under certain settings.
- `CECStandby` may impact other devices on the CEC chain depending on your setup.

## CEC settings checklist (important)

If `on` works but `off` does not, check Kodi CEC adapter settings first:

1. Kodi path: **Settings -> System -> Input -> Peripherals -> CEC Adapter**
2. Set **Device type** to **Playback device**
3. Set **Devices to power off during shutdown** to include **TV** and **AVR**
4. Verify TV-side CEC vendor setting is enabled (Anynet+, Simplink, Bravia Sync, etc.)

In testing, setting the CEC adapter **Device type** to **Playback device** was required for reliable standby/power-off behavior.
