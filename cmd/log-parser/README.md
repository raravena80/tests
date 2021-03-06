# `kata-log-parser`

* [Logfile requirements](#logfile-requirements)
* [Component logfiles](#component-logfiles)
* [Usage](#usage)

`kata-log-parser` is a tool that combines logfiles generated by the various
system components, sorts them by timestamp, and re-displays the log entries. A
time delta is added to show how much time has elapsed between each log entry.

The tool also checks the validity of all log records, can re-format the logs,
and output them in a different format.

For more information on the `kata-log-parser` tool, use the help command:

```
$ kata-log-parser --help
```

## Logfile requirements

The tool reads logfiles in the [logfmt](https://brandur.org/logfmt) structured
logging format. For example, a logfile created by the golang
[logrus](https://godoc.org/github.com/sirupsen/logrus) package.

The tool requires that the following fields are defined for each log record:

- Log level field (`level`): must be one of the logrus `LogLevel` values
  in string format (e.g. `debug`, `info`, `error`).

- Name field (`name`): a single word that specifies the name of the
  application that generates the log record (e.g. `kata-runtime`).

- Process ID field (`pid`): the numeric process identifier for the process
  that generates the log record.

- Source field (`source`): a single word that specifies the name of a unique
  part of the system (e.g. `proxy`, `runtime`, `shim`).

- Timestamp field (`time`): in [RFC3339](https://www.ietf.org/rfc/rfc3339.txt)
  format and including a nanosecond value.

Additional to the fields above, the tool also expects the following field:

- Message field (`msg`): a textual message allowing log records to be
  disambiguated.

## Component logfiles

The primary logfiles the tool reads are:

- The [proxy](https://github.com/kata-containers/proxy) log.

  This log also includes embedded log entries from the
  [agent](https://github.com/kata-containers/agent). `kata-log-parser`
  automatically unpacks and displays the entries. During this process, the
  encapsulating proxy log messages are discarded.

- The [runtime](https://github.com/kata-containers/runtime) log.

  This log also includes
  [virtcontainers](https://github.com/containers/virtcontainers) log entries.

- The [shim](https://github.com/kata-containers/shim) log.

## Usage

To merge all logs:

1. [Enable full debug](https://github.com/kata-containers/documentation/wiki/Developer-Guide#enable-full-debug).
1. Clear the systemd journal (optional):
   ```
   $ sudo systemctl stop systemd-journald
   $ sudo rm -f /var/log/journal/*/* /run/log/journal/*/*
   $ sudo systemctl start systemd-journald   
   ```
1. Create a container.
1. Collect the logs.
    1. Save the proxy log, which includes agent log details:
       ```
       $ sudo journalctl -q -o cat -a -t kata-proxy > ./proxy.log
       ```
    1. Save the runtime log:
       ```
       $ sudo journalctl -q -o cat -a -t kata-runtime > ./runtime.log
       ```
    1. Save the shim log:
       ```
       $ sudo journalctl -q -o cat -a -t kata-shim > ./shim.log
       ```
1. Ensure the logs are readable:
   ```
   $ sudo chown $USER *.log
   ```
1. To install the program:
   ```
   $ go get -d github.com/kata-containers/tests
   $ pushd $GOPATH/src/github.com/kata-containers/tests/cmd/log-parser && make install && popd
   ```
1. To run the program:
   ```
   $ kata-log-parser proxy.log runtime.log shim.log
   ```
