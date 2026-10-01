# bambuddy-notifications

The announcements feed for [Bambuddy](https://github.com/maziggy/bambuddy): short
messages from the maintainers that appear in Bambuddy's in-app announcements inbox,
such as security fixes, breaking changes, new releases and calls for testers.

## What Bambuddy fetches, and what it doesn't send

Bambuddy downloads one file, `feed.json`, from

    https://raw.githubusercontent.com/maziggy/bambuddy-notifications/main/feed.json

at startup and every few hours after that. That is a plain GET to GitHub, the same host
Bambuddy's update check already uses. Nothing else is involved:

- no Bambuddy server is contacted, so nobody but GitHub sees the request;
- nothing about your install is sent. No ID, no version, no settings;
- whether a message applies to you (version range, beta channel, install type) is decided
  by your install, locally, from what is in the file.

You can turn it off in Bambuddy's settings. It then fetches nothing at all.

## What may be sent

- security notices and urgent bug fixes;
- breaking changes and migration notes;
- new releases;
- calls for testers.

Never advertising, and never anything that needs you to enter data. Messages are plain
text and can only link to `github.com` or `bambuddy.cool`. Bambuddy refuses any other
link.

## The file

`feed.json` is written by the maintainers' tooling, not by hand. Every change is a commit
here, so this repository's history is the full public record of every message ever sent,
edited or withdrawn.

```json
{
  "format": 1,
  "key_id": "<first 16 hex of sha256(public key)>",
  "signature": "<base64 Ed25519 signature>",
  "payload": {
    "format": 1,
    "serial": 7,
    "published_at": "2026-10-01T12:00:00Z",
    "announcements": [
      {
        "id": "a3f09c1e5b7d2468",
        "level": "important",
        "published_at": "2026-10-01T12:00:00Z",
        "expires_at": null,
        "texts": {
          "en": {"title": "...", "body": "...", "link_label": "..."},
          "de": {"title": "...", "body": "..."}
        },
        "link_url": "https://wiki.bambuddy.cool/...",
        "target": {
          "min_version": null,
          "max_version": "2.4.0",
          "channels": [],
          "install_types": []
        }
      }
    ]
  }
}
```

- **`signature`** is an Ed25519 signature over the canonical form of `payload`, which is
  `json.dumps(payload, sort_keys=True, separators=(",", ":"), ensure_ascii=False)` encoded
  as UTF-8. The public key is built into Bambuddy, which ignores a file that doesn't
  verify. A copy of this repo, or anyone in the middle, cannot put words in Bambuddy's
  mouth.
- **`serial`** only goes up. Bambuddy remembers the highest serial it has seen and ignores
  anything older, so an old file can't be re-served to bring back a withdrawn message.
- **`level`:** `info` shows in the inbox only. `important` and `critical` also show a
  banner until dismissed.
- **`texts`:** English is always present. Bambuddy shows your UI language when the message
  has it, and English otherwise.
- **`target`:** an empty list or `null` means everyone. `channels` is `stable` or `beta`.
  `install_types` is `docker`, `native`, `ha_addon` or `windows`.

To check a file yourself (needs `pip install cryptography`, plus the public key from
Bambuddy's source):

```python
import base64, json
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PublicKey

envelope = json.load(open("feed.json"))
canonical = json.dumps(envelope["payload"], sort_keys=True, separators=(",", ":"),
                       ensure_ascii=False).encode()
Ed25519PublicKey.from_public_bytes(base64.b64decode(PUBLIC_KEY)).verify(
    base64.b64decode(envelope["signature"]), canonical)  # raises if it doesn't match
print("valid")
```

Questions or concerns go to the [main repository](https://github.com/maziggy/bambuddy/issues).
