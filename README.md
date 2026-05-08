# MQTTPie — Designer Guide

MQTTPie connects your ProtoPie prototype to real-world data over MQTT. You can test your prototype with simulated sensor data or connect it to real hardware — all configured from a browser interface with no coding required.

---

## Installation

1. Open **ProtoPie Connect** → Plugins → Upload the zip file provided by your team
2. Click **Run**

A browser window opens at `http://localhost:3000` — that's your MQTTPie control panel.

> MQTTPie requires a **ProtoPie Connect Enterprise** plan.

---

## The control panel

The left side has a **Channels** section where you manage all data connections.
The right side has three tabs: **Event Log**, **MQTT Watcher**, and **Settings**.

---

## Channels

Everything in MQTTPie is a channel. A channel is a connection between an MQTT topic and a ProtoPie message ID, with a direction:

| Direction | What it does |
|-----------|-------------|
| **MQTT → PP** | Receives data from a device (or simulator) and sends it into your prototype |
| **PP → MQTT** | Receives a message your prototype sends and forwards it to a device |

**To add a channel**, click **+ Add Channel** and fill in the form.

---

### MQTT → PP channels

Use these to bring data into your prototype — from real hardware or a built-in simulator.

**Without simulate** — the channel is a passthrough. It listens for real MQTT messages on the configured topic and forwards them to your prototype. Use this when a developer's hardware is connected.

**With simulate on** — the channel generates its own data automatically, so you can test without hardware. Choose a generator type:

| Type | Description |
|------|-------------|
| **Random** | Picks a random number between Min and Max each time |
| **Cycle** | Steps through a list of values in order (e.g. `online, idle, active, offline`) |
| **Boolean** | Alternates between `true` and `false` |

Each simulating channel card shows:
- The current value being published
- An **Auto** toggle — turns on continuous publishing at a set interval
- A **Send now** button — publishes a single value immediately

---

### PP → MQTT channels

Use these when your prototype needs to send a command to a device — for example, toggling a light or setting a fan speed.

Add a **Send** interaction in your Pie with the `messageId` you configured in the channel. MQTTPie will automatically forward it to the MQTT topic. The channel card shows the last value your prototype sent.

No controls appear on PP → MQTT cards — they work passively in the background.

---

### Adding and editing channels

Click **+ Add Channel** to open the configuration panel. The available fields depend on the direction and whether simulate is on.

| Field | What it means |
|-------|--------------|
| Direction | MQTT → PP or PP → MQTT |
| Label | A friendly name shown on the card |
| MQTT Topic | The topic to subscribe to or publish on (e.g. `sensors/temperature`) |
| Message ID | The ProtoPie message ID — leave blank to use the last part of the topic automatically (e.g. `sensors/temperature` → `temperature`) |
| Simulate | MQTT → PP only — turn on to generate data without hardware |
| Unit | Display unit, e.g. `°C` or `%` (optional, simulate only) |
| Type | How values are generated (simulate only) |
| Interval ms | How often to publish when Auto is on (simulate only) |

Hover a card to reveal the **edit (✎)** and **delete (×)** buttons.

> **Tip:** Switch to the **MQTT Watcher** tab to see all live traffic. Hover any row and click **+ Channel** to open the add panel pre-filled with that topic.

---

## Setting up your Pie

### Receiving data from MQTTPie

Add a **Receive** interaction in your Pie with the message ID matching your channel:

```
Receive → messageId: "temperature" → assign to variable
```

Whenever MQTTPie receives a value on that channel (simulated or from hardware), your Pie will receive it.

### Sending data from your Pie to MQTT

Add a **Send** interaction in your Pie, and create a matching **PP → MQTT** channel in MQTTPie with the same message ID:

```
Send → messageId: "lightSwitch" → value: "on"
```

MQTTPie will forward it to the MQTT topic you configured.

---

## Settings

Click the **Settings** tab on the right panel.

| Setting | Description |
|---------|-------------|
| **Built-in broker** | Default. MQTTPie runs its own MQTT broker — nothing extra to set up. Use this when testing with simulators or connecting hardware directly to your computer. |
| **External broker** | Connect to a broker on your network or the internet. Use this when a developer has set up a shared broker for the project. Enter the broker URL they provide (e.g. `mqtt://192.168.1.10:1883`). |
| **ProtoPie Connect URL** | Where Connect is running. Default is `http://localhost:9981` — only change this if Connect is on a different machine. |

Click **Save & Reconnect** after making changes.

---

## Working with real hardware

### Receiving data from hardware (MQTT → PP)

When a developer has connected real devices to a shared MQTT broker:

1. Ask them for the broker URL (e.g. `mqtt://192.168.1.10:1883`)
2. Go to **Settings** → switch to **External broker** → paste the URL → **Save & Reconnect**
3. Add the device topics as **MQTT → PP channels** — the developer will tell you which topics their devices publish to — leave **Simulate** off
4. You can disable or delete simulating channels for those topics since real data is coming in

Switch back to **Built-in** at any time to return to simulator mode.

### Sending commands to hardware (PP → MQTT)

Your prototype can also control hardware — for example, toggling a light, changing a fan speed, or sending any value to a connected device.

1. Ask the developer which **MQTT topic** the device listens on and what values it accepts (e.g. topic `home/light/switch`, values `on` / `off`)
2. In MQTTPie, click **+ Add Channel**, set the direction to **PP → MQTT**, and fill in:
   - **Message ID** — a name you'll use in your Pie (e.g. `lightSwitch`)
   - **MQTT Topic** — the topic the developer gave you (e.g. `home/light/switch`)
3. In your Pie, add a **Send** interaction wherever you want to trigger the hardware:
   ```
   Send → messageId: "lightSwitch" → value: "on"
   ```
4. When that interaction fires in your prototype, MQTTPie instantly forwards it to the device

The channel card shows the last value your prototype sent, and the **Event Log** will show a `↓ PP` entry each time it fires — useful for confirming the signal is going out.

> **Tip:** You can combine both directions. For example, a light switch prototype might receive the current light state (`MQTT → PP`) and also send toggle commands back to the hardware (`PP → MQTT`).

### Connecting an Arduino (or any hardware) directly

If you have an Arduino or similar device on the same WiFi network as your computer, you can connect it directly to MQTTPie's built-in broker — no developer setup or external broker required.

1. Find your computer's local IP address:
   - **Mac:** System Settings → Wi-Fi → Details → IP Address (e.g. `192.168.1.50`)
   - **Windows:** Start → `cmd` → type `ipconfig` → look for IPv4 Address
2. In your Arduino sketch (or device firmware), point the MQTT broker to that IP on port `1884`:
   ```
   broker: 192.168.1.50
   port:   1884
   ```
3. Keep MQTTPie on **Built-in broker** — no need to switch to External
4. Make sure your computer and the Arduino are on the **same WiFi network**

Once connected, the device's topics will appear in the **MQTT Watcher** tab. Click **+ Channel** on any row to add it.

> **Note:** Your computer's local IP can change when you reconnect to WiFi. If the Arduino stops connecting, check whether the IP has changed and update the sketch accordingly.

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

**My Pie sends a message but nothing happens on the device**
Check that you have a **PP → MQTT** channel configured with the exact `messageId` your Pie sends. The **Event Log** will show a `↓ PP` entry each time it fires — if you don't see one, the message ID in your Send interaction may not match the channel.

**Arduino isn't connecting to the built-in broker**
Make sure your computer and Arduino are on the same WiFi network, and double-check the IP address in your Arduino sketch — it must be your computer's current local IP, not `localhost`.