# Reading and Writing Data at `/ndp/public/access_testing`

This is a short guide to getting, putting, and syncing data with the `pelican` command line executable. `/ndp/public` is a Pelican namespace served through the OSDF federation, and `/access_testing` is a subdirectory. Reads can be done by any user, and writes require you to authenticate with your CILogon identity.

## When you start

Confirm the `pelican` command line executable works:

```bash
pelican --version
```

Because this namespace is part of the OSDF, use the `osdf://` URL scheme. Objects are addressed as:

```
osdf:///<namespace>/<path>
```

Note the three slashes: `osdf://` followed by the absolute namespace path. For this namespace a full object URL looks like:

```
osdf:///ndp/public/access_testing/example.txt
```

## Authenticating (CILogon)

`/ndp/public/access_testing` is publicly readable and write-protected. So you can read there, but the first time you write there, Pelican starts an OpenID-Connect login flow through CILogon. You don't run a separate login command, just issue the `get`/`put`/`sync` command and Pelican will:

1. Create a local token wallet, prompting you for a wallet password
2. Print a URL. Open it in a browser and log in with your institutional identity via CILogon
3. Fetch the resulting token automatically and continue the transfer

The token is cached in the wallet, so subsequent commands should reuse it until it expires.

## Read a file (`get`)

Download an object to a local path. Syntax is `pelican object get <source-url> <local-destination>`:

```bash
pelican object get \
  osdf:///ndp/public/access_testing/example.txt \
  ./example.txt
```

Use `.` as the destination to keep the original filename in the current directory. To download a whole directory, use `pelican object sync` (see below).

## Write a file (`put`)

Upload a local file to the namespace. Syntax is `pelican object put <local-file> <destination-url>`. Note the argument order is the reverse of `get`:

```bash
pelican object put \
  ./results.csv \
  osdf:///ndp/public/access_testing/results.csv
```

All writes require a valid token, so if you haven't authenticated yet this is where the CILogon flow above kicks in. To upload a whole directory tree, use `pelican object sync` (see below).

## Sync a directory (`sync`)

`sync` transfers a directory and only moves objects that are missing or out of date at the destination. This makes it the right choice for starting a transfer, keeping a folder up to date, or resuming an interrupted transfer. Syntax is `pelican object sync <source> <destination>`, and it works in either direction.

Push a local directory up to the namespace:

```bash
pelican object sync \
  ./local_data \
  osdf:///ndp/public/access_testing/local_data
```

Pull the namespace directory down to a local folder:

```bash
pelican object sync \
  osdf:///ndp/public/access_testing/local_data \
  ./local_data
```

## Quick reference

| Action | Command |
| --- | --- |
| Read a file | `pelican object get <src-url> <local-dest>` |
| Write a file | `pelican object put <local-file> <dest-url>` |
| Read or write a directory | `pelican object sync <src> <dest>` |
| Preview a sync | `pelican object sync --dry-run <src> <dest>` |

Where every `<...-url>` is of the form `osdf:///ndp/public/access_testing/<path>`.

## References

- [Pelican's Command Line Executable](https://docs.pelicanplatform.org/getting-data-with-pelican/client)
- [Working with Protected Data](https://docs.pelicanplatform.org/getting-data-with-pelican/auth)
- [`pelican object get`](https://docs.pelicanplatform.org/commands-reference/pelican/object/get)
- [`pelican object put`](https://docs.pelicanplatform.org/commands-reference/pelican/object/put)
- [`pelican object sync`](https://docs.pelicanplatform.org/commands-reference/pelican/object/sync)

## Installation

The `pelican` command line exectuable is already installed in the National Data Platform JupyterHub environment. But if you need to install the executable separately, this can be done like:

```bash
wget -O - "https://dl.pelicanplatform.org/latest/pelican_$(uname -s)_$(uname -m).tar.gz" | tar zx -C ~/.local/bin/ --strip-components=1
```

Make sure `~/.local/bin` is in the `$PATH` environment variable.

## Troubleshooting

If you receive an authorization error while running the `pelican` command line executable, this can sometimes be caused by stale credentials. Two troubleshooting actions are:

```bash
# First: remove the local pelican cache
rm -rf ~/.config/pelican/
# Then rerun the pelican command
# Second: open the browser link in a private/incognito browser to avoid stale cookies
```

