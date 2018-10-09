# Running applications

Starting Kore applications is done in one or two ways:

* Using _kodev run_
* Using _kore -c config_

The first method will keep the process in the foreground allowing you to shut it down using CTRL-C.

This method is aimed at developing.

The second method will read the configuration file passed on the command line, load in all required application modules and attempt to change root and drop privileges accordingly.

This method is aimed at running in production.

You can skip chroot\(\) and privdrops using -n and -r.

### Random

**Important** If you are running Kore chrooted and privilege seperated (which
you **should** be doing production), Kore will require /dev/urandom to be
created under the chroot environment for both the keymgr and worker processes.

Failing to do so will prevent your application from working.

### Halting applications

When you wish to shutdown your application gracefully you can send a SIGQUIT or SIGTERM signal to the parent process. You can find the PID for this parent process in the pidfile you specified in your configuration.

