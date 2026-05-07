# MQTTPie — Designer Guide

MQTTPie connects your ProtoPie prototype to real-world data over MQTT. You can test your prototype with simulated sensor data or connect it to real hardware — all configured from a browser interface with no coding required.

---

## Installation

1. Download the MQTTPie plugin file for your platform:
   - `plugin-bundle-macos-arm64` — Mac (Apple Silicon / M1, M2, M3)
   - `plugin-bundle-macos-x64` — Mac (Intel)
   - `plugin-bundle-win-x64.exe` — Windows
2. Zip the plugin file together with `metadata.json`
3. Open **ProtoPie Connect** → Plugins → Upload the zip

Once installed, click **Run** in Connect. A browser window opens at `http://localhost:3000` — that's your MQTTPie control panel.

> MQTTPie requires a **ProtoPie Connect Enterprise** plan.

---

## The control panel

The left side has two sections: **Simulators** and **Channels**.
The right side has three tabs: **Event Log**, **MQTT Watcher**, and **Settings**.

---

## Simulators

Simulators generate fake device data and send it automatically so you can test your prototype without real hardware.

Each simulator card shows:
- The current value being published
- The MQTT topic it publishes to
- An **Auto** toggle — turns on continuous publishing at a set interval
- A **Send now** button — publishes a single value immediately

**To add a simulator**, click **+ Add Simulator** and fill in:

| Field | What it means |
|-------|--------------|
| Label | A friendly name shown on the card |
| Topic | The MQTT topic to publish to (e.g. `sensors/temperature`) |
| Unit | Display unit, e.g. `°C` or `%` (optional) |
| Type | How values are generated — see below |
| Interval ms | How often to publish when Auto is on |

**Types:**
- **Random** — picks a random number between Min and Max each time
- **Cycle** — steps through a list of values in order (e.g. `online, idle, active, offline`)
- **Boolean** — alternates between `true` and `false`

Hover a card to reveal the **edit (✎)** and **delete (×)** buttons.

---

## Channels

Channels define which MQTT messages get forwarded to your ProtoPie prototype.

Each channel maps an **MQTT topic** → **ProtoPie message ID**. When a message arrives on that topic, MQTTPie sends it to your prototype as a `ppMessage`.

**To add a channel**, fill in the topic and message ID at the bottom and click **Add**. Leave the message ID blank to use the last part of the topic automatically (e.g. `sensors/temperature` → `temperature`).

**To send a value manually**, click on a channel row to expand it, type a value, and press **Send** or hit Enter.

Channels are saved automatically when you add or remove them.

> **Tip:** Switch to the **MQTT Watcher** tab to see all live traffic. Hover any row and click **+ Channel** to add it instantly.

---

## Setting up your Pie

### Receiving data from MQTTPie

In your Pie, add a **Receive** interaction with the message ID matching your channel:

```
Receive → messageId: "temperature" → assign to variable
```

Whenever MQTTPie sends a value on that channel, your Pie will receive it.

### Sending data from your Pie to MQTT

Add a **Send** interaction in your Pie:

```
Send → messageId: "light_switch" → value: "on"
```

MQTTPie will pick this up and publish it to the corresponding MQTT topic.

---

## Settings

Click the **Settings** tab on the right panel.

| Setting | Description |
|---------|-------------|
| **Built-in broker** | Default. MQTTPie runs its own MQTT broker — nothing extra to set up. Use this when testing with simulators. |
| **External broker** | Connect to a broker on your network or the internet. Use this when working with a developer's real hardware setup. Enter the broker URL they provide (e.g. `mqtt://192.168.1.10:1883`). |
| **ProtoPie Connect URL** | Where Connect is running. Default is `http://localhost:9981` — only change this if Connect is on a different machine. |

Click **Save & Reconnect** after making changes.

---

## Working with real hardware

When a developer has connected real devices to an MQTT broker:

1. Ask them for the broker URL (e.g. `mqtt://192.168.1.10:1883`)
2. Go to **Settings** → switch to **External broker** → paste the URL → **Save & Reconnect**
3. Add the relevant topics as **Channels** — the developer will tell you which topics their devices publish to
4. You can disable or delete your simulators for those topics since real data is coming in

Switch back to **Built-in** at any time to return to simulator mode.

---

## Troubleshooting

**ProtoPie badge shows "offline"**
Make sure ProtoPie Connect is running before starting MQTTPie. If Connect is on a different machine, update the Connect URL in Settings.

**MQTT badge shows "offline"**
In Built-in mode this shouldn't happen — try restarting the plugin. In External mode, check that the broker URL is correct and the broker is reachable from your machine.

**Prototype isn't receiving values**
Check the **Channels** section — make sure the message ID in your channel matches exactly what your Pie listens for (case-sensitive). The **Event Log** will show `↑ MQTT` entries when values are being published.

**Values are publishing but nothing happens in the Pie**
Open the **Event Log** and look for `↑ MQTT` entries. If you see them, the issue is in the Pie — check your Receive interaction's message ID spelling.