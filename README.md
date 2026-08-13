![logo](custom_components/tuya_local/brand/icon.svg)

This is a Home Assistant integration to support devices running Tuya firmware without going via
the Tuya cloud, over WiFi and (with limitations) via hubs. This fork exists to carry two patches
upstream declined; see "Why this fork exists" below. For general bugs and device requests, report
them upstream at [make-all/tuya-local](https://github.com/make-all/tuya-local/issues); this fork
does not accept unrelated contributions.

---

## Why this fork exists

This is a fork of [make-all/tuya-local](https://github.com/make-all/tuya-local): nearly all of the
code comes from that project. It exists to keep two fixes running that the upstream maintainer has
declined to merge, for devices this fork's author actually owns:

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

After installing, configure devices the same way as upstream: Settings, Devices & Services, Add
Integration, Tuya Local. For everything else, configuration, supported devices, hub setup,
protocol/connection troubleshooting, secure locks, IR/RF blasters, this fork behaves identically to
upstream, so refer to
[make-all/tuya-local's README](https://github.com/make-all/tuya-local#readme) and
[DEVICES.md](https://github.com/make-all/tuya-local/blob/main/DEVICES.md).
