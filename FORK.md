# This fork

`Shoot-and-Destroy/enet` is [lsalzman/enet](https://github.com/lsalzman/enet)
plus what our game server needs:

- **IPv6** — the implementation of [lsalzman/enet#109](https://github.com/lsalzman/enet/pull/109)
  by Jérôme Leclercq (SirLynix), kept current (his `ipv6` branch, merged
  2026-11-05). Hosts take `ENET_ADDRESS_TYPE_IPV4`, `ENET_ADDRESS_TYPE_IPV6`
  or `ENET_ADDRESS_TYPE_ANY` (one dual-stack socket); `ENetAddress` holds
  either family in a union. The wire protocol is untouched
  (`include/enet/protocol.h` is identical to upstream), so a peer built
  without the patch connects to an IPv4 or a dual-stack host as before.
  Rebases are offered upstream as [lsalzman/enet#287](https://github.com/lsalzman/enet/pull/287).
- **srtt rounding** — [lsalzman/enet#162](https://github.com/lsalzman/enet/pull/162):
  a measured rtt under 8 ms never lowered the srtt (the EWMA truncated
  `diff / 8`), so a loopback host read 8 ms forever. `protocol.c` rounds
  the adjustments up.
- **getsockname length** — `enet_socket_get_address` passed an uninitialized
  `int` as the address length (the IPv4-only code initialized its
  `socklen_t`; the IPv6 rewrite lost it). `unix.c` uses `socklen_t` now.

## Keeping it current

`.github/workflows/upstream.yml` runs every Monday and on demand: it merges
`lsalzman/enet`'s `master` into ours and opens the `upstream-sync` pull
request. `cmake.yml` builds it on Linux, macOS and Windows.

**Do not use GitHub's "Sync fork" button.** This repository is a fork of
`SpadesX/enet`, which is itself a fork of `lsalzman/enet`: "Sync fork"
would sync from SpadesX, which has not moved since February 2024, and would
move `master` backwards. Use the workflow, or by hand:

```sh
git remote add upstream https://github.com/lsalzman/enet.git   # once
git fetch upstream master
git merge upstream/master
```

The recurring conflict is `win32.c`'s include block: the IPv6 code needs
`memory.h`, `stdio.h`, `string.h`, `ws2tcpip.h` and `ws2ipdef.h`. Check that
`git diff upstream/master -- include/enet/protocol.h` stays empty after a
merge; the wire format must not change.

After a merge here, the game repository bumps its `common/enet` submodule to
the new `master` (`make enet-update`, then commit the pointer).
