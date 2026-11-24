# Verified Deco Wi-Fi writes

This development branch adds `TPLinkDecoClient.set_wifi_verified()` while retaining the existing `set_wifi()` API. The new method sends one write to the existing WLAN endpoint and then performs a limited number of read requests. Timeout, session error, and unreadable response handling never replay the write.

The return value contains only recognized Wi-Fi states. A parsed router rejection, an observed state that differs from the requested value, and an unavailable verification produce distinct sanitized exceptions. Normal polling uses the same strict conversion of Wi-Fi values, including `off` and `false`.

The classic Deco transport exposes `supports_verified_wifi = True`. Regional SG and E4R transports expose `False` and retain their existing control path in the companion integration.

## Compatibility status

The endpoint and payload remain those of the upstream library. Unit tests validate control flow, state interpretation, isolation of the requested band, and failure handling. They do not establish that a particular BE65 firmware accepts guest Wi-Fi writes.

Hardware validation requires a baseline of all host and guest bands, one guest-only write, a fresh read, and an independent check in the official Deco application. Verification must include the unaffected bands.

## Source and distribution

This work is based on upstream version `5.42.0`, commit `4f6995b7862d15ab391775af9af597962257f754`. The existing GPLv3 license and upstream notices remain applicable. Installation examples must not contain real credentials, network identifiers, or router responses.
