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
so only the job's own machine can reach it.

## Inputs

| Input | Default | What it does |
| --- | --- | --- |
| `connect` | | `free`, or one proxy line. Empty: install only. |
| `proxies-file` | | A file with one proxy per line, used in turn. |
| `rotate` | `every` | With `proxies-file`: `every`, `every:N`, `every:Nm` (minutes), `random`, `on-failure`. |
| `country` | | With `connect: free`: a two-letter country. |
| `listen` | `127.0.0.1:8080` | Where the proxy listens. |
| `export-proxy` | `true` | Set `HTTP_PROXY`, `HTTPS_PROXY` and `ALL_PROXY` for the steps that follow. |

## Outputs

| Output | What it is |
| --- | --- |
| `version` | The hproxy version installed. |
| `proxy-url` | The proxy's address, when one was started. |

## License

MIT. [hproxy.com](https://hproxy.com)
