# Verified Deco Wi-Fi writes

This development branch adds `TPLinkDecoClient.set_wifi_verified()` while retaining the existing `set_wifi()` API. Without additional configuration, the new method sends one write to the existing WLAN endpoint and then performs a limited number of read requests. Timeout, session error, and unreadable response handling never replay the write.

The return value contains only recognized Wi-Fi states. A parsed router rejection, an observed state that differs from the requested value, and an unavailable verification produce distinct sanitized exceptions. Normal polling uses the same strict conversion of Wi-Fi values, including `off` and `false`.

The classic Deco transport exposes `supports_verified_wifi = True`. Regional SG and E4R transports expose `False` and retain their existing control path in the companion integration.

## Optional local app transport

Install the library with its `deco-tmp` extra to enable the optional Paramiko dependency. A classic Deco client can opt in through `configure_tmp_wifi(username_sha1=..., host_key_sha256=...)`. Both values must be supplied explicitly. Configuration validates the values locally and does not connect to the router.

The account username is the SHA1 identifier used by the Deco app transport. The web interface's `admin` session name does not identify that account. The existing client password is reused. The SSH host key is checked against the supplied SHA256 fingerprint before a single password authentication attempt.

Guest controls then use local SSH port 20001, a fixed loopback TMP channel, and the app's WLAN read/write commands. Requests contain only the requested guest band's `enable` field. A fresh read of an already matching state avoids an unnecessary write. Otherwise, one write is followed by bounded reads. A lost app connection permits read-only verification through the already authorized HTTP session; it never triggers another SSH authentication or an HTTP write fallback. Host Wi-Fi and other transports retain their previous behavior.

The TMP framing is documented by [pydeco-tmp](https://github.com/mthsg/pydeco-tmp). The WLAN opcode mapping was checked against the corresponding command table in the [official BE65 V2 firmware](https://www.tp-link.com/en/support/download/deco-be65/v2/). This branch implements only WLAN reads and guest-band writes from that protocol.

## Compatibility status

The default HTTP endpoint and payload remain those of the upstream library. The optional app transport uses a separate authenticated protocol. Unit tests validate framing, control flow, state interpretation, isolation of the requested band, and failure handling. Passing unit tests does not establish that a particular firmware accepts guest Wi-Fi writes.

Hardware validation requires a baseline of all host and guest bands, one guest-only write, a fresh read, and an independent check in the official Deco application. Verification must include the unaffected bands.

## Source and distribution

This work is based on upstream version `5.42.0`, commit `4f6995b7862d15ab391775af9af597962257f754`. The existing GPLv3 license and upstream notices remain applicable. Installation examples must not contain real credentials, network identifiers, or router responses.
