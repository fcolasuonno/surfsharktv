# surfsharktv

A self-contained Surfshark WireGuard client for Tizen TVs (or any Linux), built on userspace
[wireguard-go](https://git.zx2c4.com/wireguard-go). It creates a `surftun` interface, sets up wg-quick-style policy
routing, forces DNS to Surfshark's resolvers, blocks IPv6 egress and can enable a kill switch.

There is **no account login**. Generate a WireGuard key pair in Surfshark's *Manual setup → Router → WireGuard*
page, paste the public key into the website, and give the matching private key to this program. For each start it
reads Surfshark's public cluster list (`https://api.surfshark.com/v4/server/clusters/generic`) and selects the least
loaded server in the requested country. No credentials, tokens or account data are sent anywhere.

This is the helper binary used by the [`surfshark`](https://github.com/andrearicci93/vetro-plugins/tree/main/plugins/surfshark)
Vetro Shard. The Shard drives it through `ctx.execute` as root.

## Commands

```
surfsharktv up   [-key keyfile | -conf file.conf] [-country CODE] [-server HOST] [-killswitch]
surfsharktv down
surfsharktv status
surfsharktv countries

managed mode (settings in /opt/surfsharktv/config):
surfsharktv start [CC] [--no-ks|--ks]   start in background
surfsharktv stop | restart
surfsharktv set country CC | set killswitch on|off
surfsharktv log [N]
surfsharktv state | status -json        machine-readable state
surfsharktv verify                      check the saved private key or WireGuard config
surfsharktv uninstall
```

State lives in `/opt/surfsharktv` (folder `0700`, files `0600`, binary `0700`): `config`, `wg.key` (private key you
registered on the website), `wg.conf` (a full config instead), `log`, and the running copy is staged at
`/opt/surfsharktv/ro/surfsharktv`.

## Build

Any Go version that satisfies `go.mod` (currently 1.26.0) builds it:

```sh
./build.sh                 # -> /tmp/surfsharktv-armv7 and /tmp/surfsharktv-armv7.gz, with its SHA-256
```

The output is a static ARMv7 ELF, gzipped for distribution. Pin the printed SHA-256 in the Shard's `index.js` before
publishing a release.

## Tizen UEP

Newer Tizen firmware enforces UEP, which refuses to execute unsigned ELFs on writable filesystems but skips files on
**read-only VDFS** mounts. The Shard therefore bind-mounts `/opt/surfsharktv/ro` onto itself, remounts it read-only,
and runs the binary from there. Running the binary from `/opt/surfsharktv` or `/dev/shm` directly will fail with
`Operation not permitted` on those TVs.

## License

MIT — see [LICENSE](LICENSE). Third-party notices are in [NOTICE](NOTICE).
