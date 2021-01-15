# cronwrapper

A wrapper for cron jobs.

Cron normally mails all output of a job to the job's owner. This can be
very noisy: A job which runs every minute can produce up to 1440 mails
per day. A common technique is to redirect stdout and/or stderr to a
file or even /dev/null, but that may be too silent: Important error
messages are quietly filed away and nobody notices them.

This program is intended to strike the goldilocks zone: All output is
written to a log to be reviewed when necessary. In addition, success and
failure is indicated by the presence or absence of flag files which can
be tested for by a monitoring tool (a check script for nagios/icinga is
included.). What constitutes success or failure can also be configured
by specifying patterns to be matched against stdout, stderr or the
return code.

## Status files

If the job finished successfully, cronwrapper creates a file `ok` in the
output directory and deletes the file `fail`. If the job finished
unsuccessfully, it creates `fail` and deletes `ok`. So the existence or
non-existence of these two files can be used to check the status of the
most recent run.

A third file `fail_sticky` is also created on error but never deleted
automatically. This is intended to signal transient errors. If the file
exists, the job failed at some point in the past and the system
administrator should investigate the issue and remove `fail_sticky`
after resolving the issue.

## Invocation

In simple cases `cronwrapper` can be invoked with a program and its
arguments on the command line. This will simply invoke the program with
the given arguments and record the result in a subdirectory of
`$HOME/log/cronwrapper`. The job is considered successful if it
terminates with and exit code of 0 and there was no output on stderr.

If more complex configuration is needed, `cronwrapper` is invoked with a
directory as its only argument. The directory must contain a file
`config.json` and is also used for log and status files.

## Configuration

The configuration file is – as the name `config.json` suggests – in JSON
format and contains a single object with these fields:

|   |   |
|---|---|
| `command` | A list with the command to invoke and its arguments |
| `err`, `out`, `rc` | A list of filter rules for stderr, stdout and the exit code (aka return code). Every rule consists of a regular expression and a result (one of `"OK"`, `"FAIL"`, `"OK_STICKY"`, `"FAIL_STICKY"` in increasing priority). For each line of output (and the exit code) the corresponding filter list is checked and the first updates the status if the new status has higher priority than the previous status. |
| `passthrough_stdout` | Wenn dieses Feld einen wahren Wert hat, wird stdout nicht nur geloggt, sondern auch ausgegeben. |

### Examples

```
{
    "command": [ "/home/wdsimp/bin/import_OECD_NAQ" ]
}
```

Simply invokes `/home/wdsimp/bin/import_OECD_NAQ` with the default
filter rules. Equivalent to invoking `cronwrapper
/home/wdsimp/bin/import_OECD_NAQ` except that the latter puts logs and
status files into `$HOME/log/cronwrapper/import_OECD_NAQ` while the
former puts them into the same directory where the `consfig.json` is.

```
{
    "command": "/home/qpsmtpd/bin/git_update_qpsmtpd",
    "err": [
        [ "^From git", "OK" ],
        [".*master +-> origin/master", "OK"],
        ["^.*", "FAIL"]
    ]
}
```

This uses non-standard rules for stderr. The outputs matching the regegs
`^From git` or `.*master +-> origin/master` are considered ok, all other
output on stderr signifies failure.

```
{
    "command": [ "/usr/local/sbin/watch_patroni" ],
    "passthrough_stdout": true
}
```

The script `watch_patroni` only prints to stdout if something notable
happened. We want cron to send an email in this case, so we set
`passthrough_stdout`, which will pass stdout through to cron which will
then send an email to the crontab owner. (Alternatively, `watch_patroni`
could be rewritten to send an email instead of printing to stdout)
