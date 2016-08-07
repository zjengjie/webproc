# webproc

Wrap any program in a simple web-based user-interface

## Install

**Binaries**

See [the latest release](https://github.com/jpillora/webproc/releases/latest) or download it now with `curl https://i.jpillora.com/webproc | bash`

**Source**

``` sh
$ go get -v github.com/jpillora/webproc
```

## Usage

Let's use `webproc` to run `dnsmasq`:

```
webproc --config /etc/dnsmasq.conf -- dnsmasq --no-daemon
```

Visit http://localhost:8080 and view the process configuration, status and logs.

**SCREENSHOT**

Bonus, we can add configuration validation if your process supports it:

```
webproc --config /etc/dnsmasq.conf --verify 'dnsmasq --test' -- dnsmasq --no-daemon
```

## CLI

```
$ webproc --help

  Usage: webproc [options] args...

  args can be either a command with arguments or a webproc file

  Options:
  --host, -h    listening interface
  --port, -p    listening port
  --user, -u    basic auth username
  --pass        basic auth password
  --config, -c  comma-separated list of configuration files
  --verify, -v  command used to verify configuration
  --help
  --version

  Version:
    0.0.0-src

  Read more:
    https://github.com/jpillora/webproc

```

## Config

The CLI interface only exposes a subset of the configuration, to further customize
webproc, create a `program.toml` file and then load it with:

```
webproc program.toml
```

Here is a complete configuration with the defaults, only `ProgramArgs` is **required**:

[embedmd]:# (default.toml)
```toml
# Interface to serve web UI. Warning: defaults to ALL interfaces.
Host = "0.0.0.0"

# Port to serve web UI
Port = 8080

# Basic authentication settings for web UI
User = ""
Pass = ""

# List of IP addresses (and optional subnets) allowed to access the web UI.
# For example, allow 192.168.0.X with "192.168.0.0/24"
AllowedIPs = []

# Program to execute (with optional Arguments). Note: the process
# must remain in the foreground (i.e. do NOT fork/run as daemon).
ProgramArgs = []

# Log settings for the process:
# "both" - log to both, webproc standard out/error and to the web UI log.
# "webui" - only log to the web UI. Note, the web UI only keeps the last 10k lines.
# "proxy" - only log to webproc standard out/error.
Log = "both"

# OnExit dictates what action to take when the process exits:
# "proxy" - also exit webproc with the same exit code
# "ignore" - ignore and wait for manual restart via the web UI
# "restart" - automatically restart with exponential backoff time delay between failed restarts
# It is recommended to use "proxy" and then to run webproc commands via a process manager.
OnExit = "proxy"

# Configuration files to be editable by the web UI.
# For example, dnsmasq would include "/etc/dnsmasq.conf"
ConfigurationFiles = []

# When provided, this command will be used verify all configuration changes
# before restarting the process. An exit code 0 means valid, otherwise it's assumed invalid.
VerifyProgramArgs = []

# When provided, this signal is used to restart the process. It's set to interrupt (SIGINT) by default, though
# some programs support zero down-time configuration reloads via SIGHUP, SIGUSR2, etc.
RestartSignal = "SIGINT"

# After the restart signal has been sent, webproc will wait for RestartTimeout before
# forcibly restarting the process.
RestartTimeout = "30s"
```