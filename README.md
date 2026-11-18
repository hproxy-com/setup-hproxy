# Set up hproxy

A GitHub Action that installs the [hproxy](https://github.com/hproxy-com/proxy-all-in-one-tool)
command-line tool on the job's machine (Linux x64 and ARM, macOS, Windows), and starts a rotating
proxy for the rest of the job if you like.

## Install the tool

```yaml
- uses: hproxy-com/setup-hproxy@v1
- run: hproxy list --protocol socks5 --country US --limit 5
```

It installs through hproxy.com's own installers, which check the file against its SHA-256 first.

## A rotating proxy for the job

The steps after it reach the internet through the proxy: `HTTP_PROXY`, `HTTPS_PROXY` and
`ALL_PROXY` point at it.

```yaml
# The free HProxy pool, with a new exit whenever one stops answering
- uses: hproxy-com/setup-hproxy@v1
  with:
    connect: free
    country: US

# Your own proxy, its login kept in a secret
- uses: hproxy-com/setup-hproxy@v1
  with:
    connect: ${{ secrets.PROXY_LINE }}

# Your own list, the next proxy for every connection
- uses: hproxy-com/setup-hproxy@v1
  with:
    proxies-file: proxies.txt
    rotate: every
```

A proxy line can take any shape your provider prints: `host:port:user:pass`,
`http://user:pass@host:port`, `socks5://user:pass@host:port`. The proxy listens on `127.0.0.1:8080`,
so only the job's own machine can reach it. The job's own machine stays off the proxy
(`NO_PROXY` gets `localhost,127.0.0.1,::1`), so a test server the job starts stays reachable.

## Which sites use it

```yaml
# Everything through the proxy except these
- uses: hproxy-com/setup-hproxy@v1
  with:
    connect: ${{ secrets.PROXY_LINE }}
    skip: github.com,githubusercontent.com,10.0.0.0/8

# Only these through the proxy
- uses: hproxy-com/setup-hproxy@v1
  with:
    connect: free
    only: example.com

# Two proxies in a row: yours only ever sees the first one's address
- uses: hproxy-com/setup-hproxy@v1
  with:
    connect: ${{ secrets.PROXY_LINE }}
    via: ${{ secrets.FIRST_HOP }}
```

A site is a name (`example.com` covers `www.example.com` and everything else under it), an
address, or a range.

## Inputs

| Input | Default | What it does |
| --- | --- | --- |
| `connect` | | `free`, or one proxy line. Empty: install only. |
| `proxies-file` | | A file with one proxy per line, used in turn. |
| `rotate` | `every` | With `proxies-file`: `every`, `every:N`, `every:Nm` (minutes), `random`, `on-failure`. |
| `country` | | With `connect: free`: a two-letter country. |
| `socks5` | `false` | With `connect: free`: SOCKS5 exits only. |
| `skip` | | Sites that go straight out, past the proxy, separated by commas. |
| `only` | | Only these sites use the proxy. |
| `via` | | A proxy in front of the proxy. |
| `listen` | `127.0.0.1:8080` | Where the proxy listens. |
| `export-proxy` | `true` | Set `HTTP_PROXY`, `HTTPS_PROXY` and `ALL_PROXY` for the steps that follow, and keep this machine off them with `NO_PROXY`. |

## Outputs

| Output | What it is |
| --- | --- |
| `version` | The hproxy version installed. |
| `proxy-url` | The proxy's address, when one was started. |

## License

MIT. [hproxy.com](https://hproxy.com)
