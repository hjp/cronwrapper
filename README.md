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
