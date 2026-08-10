![logo](custom_components/tuya_local/brand/icon.svg) 

This fork exists to carry two patches upstream has declined (see "Why this fork exists" below).
For general bugs and device requests, report them upstream at
[make-all/tuya-local](https://github.com/make-all/tuya-local/issues); this fork does not accept
unrelated contributions. Credit for the integration itself belongs to
[Jason Rumney and the many other contributors](https://github.com/make-all/tuya-local/blob/main/ACKNOWLEDGEMENTS.md).

[![BuyMeCoffee](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://www.buymeacoffee.com/jasonrumney)

This is a Home Assistant integration to support devices running Tuya
firmware without going via the Tuya cloud.  Devices are supported
over WiFi, limited support for devices connected via hubs is available.

Note that many Tuya devices seem to support only one local connection.
If you have connection issues when using this integration, ensure that
other integrations offering local Tuya connections are not configured
to use the same device, mobile applications on devices on the local
network are closed, and no other software is trying to connect locally
to your Tuya devices.

Using this integration does not stop your devices from sending status
to the Tuya cloud, so this should not be seen as a security measure,
rather it improves speed and reliability by using local connections,
and may unlock some features of your device, or even unlock whole
devices, that are not supported by the Tuya cloud API.

A similar but unrelated integration is
[rospogrigio/localtuya](https://github.com/rospogrigio/localtuya/), if
your device is not supported by this integration, you may find it
easier to set up using that, or another more recent fork, as an alternative.

---

## Why this fork exists

This is a fork of [make-all/tuya-local](https://github.com/make-all/tuya-local): nearly all of the
code, and most of this README, come from that project. It exists to keep two fixes running that the
upstream maintainer has declined to merge, for devices this fork's author actually owns:

- **[Product id tie-breaking](https://github.com/make-all/tuya-local/issues/5854)** (closed
  `not_planned`). When two different devices share the same Tuya product id (for example, two
  different Create XW-FAN-215-D variants that both declare product id `p8z27dfdwc4riyp9`),
  `match_quality` picked whichever device config happened to be read from disk first, rather than
  the one that actually fit the device's reported data points. The fix ranks product-id matches by
  their dps fit too, so the tie is broken by which config actually matches, not by file read order.
- **[Energy sensor for the Gosund SP211](https://github.com/make-all/tuya-local/pull/4070)**
  (closed without merging). Exposes dp 17/25 as a dedicated, hidden diagnostic Energy sensor
  instead of leaving them as an opaque `add_ele`/`ele_calibration` pair. Note that on this device,
  the metering dps (current/power/voltage/energy) go silent entirely, locally and in the vendor
  app, if the device loses internet access: they appear to depend on an occasional cloud check-in
  even though `tuya_local` reads them locally. If yours are stuck on `unknown`, check whether
  anything on your network (a firewall rule, an "isolate this device" toggle, etc.) is blocking it.

Neither is a code-quality disagreement; both are device-behavior judgment calls the maintainer made
differently than this fork's author would for their own hardware. Pull request creation against
upstream has also been restricted to collaborators, so there's no path to reopen either
conversation there.

This fork tracks upstream `main` closely and carries only the two patches above on top of it. If
you don't specifically need them, use
[make-all/tuya-local](https://github.com/make-all/tuya-local) instead: it's the actively
maintained original, and new device support and bug fixes land there first. This fork is not a
general-purpose alternative and does not accept unrelated device-support contributions; it exists
solely to carry these two patches forward.

---

## Installation

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg?style=for-the-badge)](https://github.com/hacs/integration)

This fork is not in the default HACS store (it's a fork of an integration that's already listed
there under the original name), so it has to be added as a **custom repository**:

1. In Home Assistant, go to HACS.
2. Click the three-dot menu in the top right corner and select **Custom repositories**.
3. Add the repository URL: `https://github.com/smeegoan/tuya-local-alternative`
4. Set the type to **Integration** and click **Add**.
5. Find "Tuya Local Alternative" in HACS and install it like any other integration.

Or, if your Home Assistant instance is set up with [My Home
Assistant](https://www.home-assistant.io/integrations/my/), the button below attempts the same
thing in one click:

[![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=smeegoan&repository=tuya-local-alternative&category=integration)

There are no GitHub releases on this fork, so HACS installs directly from the `main` branch: that's
expected, and is how the two patches above stay included.

**Do not install this alongside the official `make-all/tuya-local`.** Both use the same
`tuya_local` integration domain, so remove one before adding the other.

## Configuration

After installing, you can easily configure your devices using the Integrations configuration UI.  Go to Settings / Devices & Services and press the Add Integration button, or click the shortcut button below (requires My Homeassistant configured).

[![Add Integration to your Home Assistant
instance.](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=tuya_local)

### Choose your configuration path

There are two options for configuring a device:
- You can login to Tuya cloud with the Tuya or SmartLife app and retrieve a list of devices and the necessary local connection data.
- You can provide all the necessary information manually [as per the instructions in DEVICES_DETAILS.md](DEVICE_DETAILS.md#finding-your-device-id-and-local-key).

The first choice essentially automates all the manual steps of the second and without needing to create a Tuya IOT developer account. This is especially important now that Tuya has started time limiting access to a key data access capability in the IOT developer portal to only a month with the ability to refresh the trial of that only every 6 months.

The cloud assisted choice will guide you through authenticating, choosing a device to add from the list of devices associated with your Tuya account, locate the device on your local subnet and then drop you into [Stage One](#stage-one) with fully populated data necessary to move forward to [Stage Two](#stage-two).

The Tuya authentication token expires after a small number of hours and so is not saved by the integration. But, as long as you don't restart Home Assistant, this allows you to add multiple devices one after another only needing to authenticate once for the first one.

### Stage One

The first stage of configuration is to provide the information needed to connect to the device.

When using the cloud assisted config, the device id and local key will be pre-filled from the cloud, and the IP address will also be filled if local discovery is not blocked by other integrations or a complex network setup. Otherwise, see [DEVICE_DETAILS.md](DEVICE_DETAILS.md) for instructions on how to find the info.

#### host

&nbsp;&nbsp;&nbsp;&nbsp;_(string) (Required)_ IP or hostname of the device.

#### device_id

&nbsp;&nbsp;&nbsp;&nbsp;_(string) (Required)_ Device ID retrieved

#### local_key

&nbsp;&nbsp;&nbsp;&nbsp;_(string) (Required)_ Local key retrieved

Note that each time you pair the device, the local key changes, so if you obtained the local key using the instructions below, then re-paired with your manufacturer's app, then the key will have changed already.

#### protocol_version

&nbsp;&nbsp;&nbsp;&nbsp;_(string or float) (Required)_ Valid options are "auto", 3.1, 3.2, 3.3, 3.4, 3.5, 3.22.  If you aren't sure, choose "auto", but some 3.2, 3.22 and maybe 3.4 devices may be misdetected as 3.3 (or vice-versa), so if your device does not seem to respond to commands reliably, try selecting between those protocol versions. Protocol 3.22 is a special case, that enables tinytuya's "device22" detection with protocol 3.3. Previously we let tinytuya auto-detect this, but it was found to sometimes misdetect genuine 3.3 devices as device22 which stops them receiving updates, so an explicit version was added to enable the device22 detection.

At the end of this step, an attempt is made to connect to the device and see if
it returns any data. For tuya protocol version 3.1 devices, the local key is
only used for sending commands to the device, so if your local key is
incorrect the setup will appear to work, and you will not see any problems
until you try to control your device.  For more recent Tuya protocol versions,
the local key is used to decrypt received data as well, so an incorrect key
will be detected at this step and cause an immediate failure.


### Stage Two

The second stage of configuration is to select which device you are connecting.
The list of devices offered will be limited to devices which appear to be
at least a partial match to the data returned by the device.

#### type

&nbsp;&nbsp;&nbsp;&nbsp;_(string) (Optional)_ The type of Tuya device.
Select from the available options.

The list presented is filtered to exclude devices that definitely do not match among the 1000+ supported devices. If a device config you expected is not shown, you may have a different firmware version, so the best way to report this is as a new device.

If you pick the wrong type, you will need to delete the device and set
it up again. This is because different types of devices create different
entities, so changing the device type without deleting everything is
not advisable.

### Stage Three

The final stage is to choose a name for the device in Home Assistant.

If you have multiple devices of the same type, you may want to change
the name to make it easier to distinguish them.

#### name

&nbsp;&nbsp;&nbsp;&nbsp;_(string) (Required)_ Any unique name for the
device.  This will be used as the base for the entity names in Home
Assistant.

---

## Device support

A list of currently supported devices can be found in the [DEVICES.md](https://github.com/make-all/tuya-local/blob/main/DEVICES.md) file.

Note that devices sometimes get firmware upgrades, or incompatible
versions are sold under the same model name, so it is possible that
the device will not work despite being listed.

Battery powered devices such as door and window sensors, smoke alarms
etc which do not use a hub are not possible to support locally, due
to the power management that they need to do to get acceptable battery
life. In some cases that may also apply when a device that can be
either battery or USB powered is plugged into USB. If you cannot gather
Warning level logs with dps listed when attempting to set it up, then it
will likely not work with this integration.

Hubs are currently supported, but with limitations.  Each connection
to a sub device uses a separate network connection, but like other
Tuya devices, hubs are usually limited in the number of connections
they can handle, with typical limits being 1 or 3, depending on the specific
Tuya module they are using.  This severely limits the number of sub devices
that can be connected through this integration.

Sub devices should be added using the `device_id`, `address` and `local_key`
of the hub they are attached to, and the `node_id` of the sub-device. If there
is no `node_id` listed, try using the `uuid` instead.

Tuya Zigbee devices are usually standard zigbee devices, so as an
alternative to this integration with a Tuya hub, you can use a
supported Zigbee USB stick or Wifi hub with
[ZHA](https://www.home-assistant.io/integrations/zha/#compatible-hardware)
or [Zigbee2MQTT](https://www.zigbee2mqtt.io/guide/adapters/).

Some Tuya Bluetooth devices can be supported directly by the
[tuya_ble](https://github.com/PlusPlus-ua/ha_tuya_ble/) integration.

Some Tuya hubs now support Matter over WiFi, and this can be used as an
alternative to this integration for connecting the hub and sub-devices
to Home Assistant. Other limitations will apply to this, so you might want
to try both, and only use this integration for devices that are not working
properly over Matter.

Tuya IR hubs that expose general IR remotes as sub devices usually
expose them as one way devices (send only) except in learning mode,
if they expose them at all locally. In general, Tuya IR hubs are only
useful for HA's built in IR support, not for any Tuya features such as their
predefined (cloud only) device database, or climate device simulation.

Want to add support for a new device, or build a device configuration file? That process, and
requesting new device support, is unchanged from upstream: see
[make-all/tuya-local's Contributing docs](https://github.com/make-all/tuya-local/blob/main/custom_components/tuya_local/devices/README.md)
and file it there, not on this fork.

---

## Offline operation issues

Many Tuya devices will stop responding if unable to connect to the
Tuya servers for an extended period.  Reportedly, some devices act
better offline if DNS as well as TCP connections is blocked.

## General issues

Many Tuya devices do not handle multiple commands sent in quick
succession.  Some will reboot, possibly changing state in the process,
others will go offline for 30s to a few minutes if you overload them.
There is some rate limiting to try to avoid this, but it is not
sufficient for some devices, and may not work across entities where
you are sending commands to multiple entities on the same device.  The
rate limiting also combines commands, which not all devices can
handle. If you are sending commands from an automation, it is best to
add delays between commands - if your automation is for multiple
devices, it might be enough to send commands to other devices first
before coming back to send a second command to the first one, or you
may still need a delay after that.  The exact timing depends on the
device, so you may need to experiment to find the minimum delay that
gives reliable results.

Most devices can handle multiple commands in a single message, so for
entity platforms that support it (eg climate `set_temperature` can
include presets, lights pretty much everything is set through
`turn_on`) multiple settings are sent at once.  But some devices do
not like this and require all commands to set only a single dp at a
time, so you may need to experiment with your automations to see
whether a single command or multiple commands (with delays, see above)
work best with your devices.

When adding devices, some devices that are detected as protocol version
3.3 at first require version 3.2 to work correctly. Either they cannot be
detected, or work as read-only if the pprotocol is set to 3.3.

## Connecting to devices via hubs

If your device connects via a hub (eg. battery powered water timers) you have to provide the following info when adding a new device:

- Device id (uuid): this is the **hub's** device id
- IP address or hostname: the **hub's** IP address or hostname
- Local key: the **hub's** local key
- Sub device id: the **actual device you want to control's** `node_id`. Note this `node_id` differs from the device id, you can find it with tinytuya as described below.

## Secure locks

Many locks are designed with basic security controls to make remote unlocking
more difficult. This integration supports the standard BLE lock model from Tuya
which uses a pair of dps (60: `remote_no_pd_seykey`, 61: `remote_no_dp_key`)
to share a key between the app and the lock during the pairing phase.
If you have access to the Tuya developer portal, you can eavesdrop on the
second of these messages when the app is used to unlock the lock remotely.
If you capture the value sent by the app, then you can decode it using a base64
decoder such as https://base64decode.org.
The format has 4 bytes of binary data, followed by an 8 digit ASCII numeric
code, followed by 3 or 4 more bytes of binary data.

The 8 digit numeric code from the first app that was paired should work for
unlocking the lock.

Although this is documented in the BLE lock documentation from Tuya, Zigbee
and WiFi locks often use the same naming for datapoints, which may be
compatible with this scheme.

## IR/RF blasters

Tuya IR and RF blasters are exposed as remote entities and support learning and
sending commands via the standard Home Assistant remote services. IR blasters are
also exposed as general `infrared` emitters, and learned commands can be sent to
other `infrared` emitters using the tuya-local specific "Send Learned IR command"
service.

### Learning commands

Use the `remote.learn_command` service with:
- `command`: the name to store the command under (e.g. `power`)
- `device`: a name for the appliance being controlled (e.g. `TV`)
- `command_type`: set to `rf` for RF remotes, omit or leave blank for IR

The integration will put the blaster into learning mode and wait up to 30 seconds
for you to press a button on the original remote. The learned code is stored
persistently and survives restarts.

### Sending commands

Using the `infrared` platform, you can send known IR commands using
other HA integrations, including
[HAIR](https://github.com/DAB-LABS/HAIR), a custom integration for
learning remote commands via an ESPHome receiver and sending them to
any supported `infrared` emitter.

To send learned commands using the same `remote` entity, you use the
`remote.send_command` service with the same `command` and `device`
values used when learning. You can also send known Tuya codes directly
without learning first:

- **IR inline code**: prefix with `b64:` followed by the base64-encoded IR code
- **RF inline code**: prefix with `rf:` followed by the base64-encoded RF code

There is also a special `send_learned_ir_command` service for sending commands
learned by the `remote` entity to any `infrared` emitter (including non-Tuya ones).
To use this, you must specify the `remote` entity the learned command was saved
with, the target `infrared` emitter entity, and the `command` and optional `device`
the command was saved as.


### UI

If you would like to expose the learnt commands as buttons in the user interface
you might want to take a look at the [Remote buttons](https://github.com/kongo09/remote_buttons)
integration, which is compatible with Tuya Local.

## Pet feeders

Many pet feeders expose an encoded **Meal plan** setting via a text entity. By default this is disabled, but you can enable it under the Device settings in HA. When enabled many pet feeders share the same underlying format, which is supported by the [FrederikM97/mealplan-card](https://github.com/FredrikM97/mealplan-card) custom card.

