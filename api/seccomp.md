# seccomp

(This is only valid for Linux).

Kore uses seccomp to filter which system calls its processes can make.

As an application developer you can extend the allow-list to better
suit your application its needs.

## Adding your own seccomp rules

If you wish to extend the allow-list, you can use the KORE_SECCOMP_FILTER
macro. In the example below we allow ioctl(2) and shmat(2) are allowed.

```
#include <kore/seccomp.h>

KORE_SECCOMP_FILTER("app",
	KORE_SYSCALL_ALLOW("ioctl"),
	KORE_SYSCALL_ALLOW("shmat")
);
```

In another example, we allow write() to stdout but no other file descriptor.

```
#include <kore/seccomp.h>

KORE_SECCOMP_FILTER("app",
	KORE_SYSCALL_ALLOW_ARG("write", 0, STDOUT_FILENO),
	KORE_SYSCALL_DENY("write", EPERM)
);
```

Kore provides a few handy macros that can be used in a KORE_SECCOMP_FILTER:

- KORE_SYSCALL_DENY(name, errno)
- KORE_SYSCALL_DENY_ARG(name, argidx, val, errno)
- KORE_SYSCALL_DENY_MASK(name, argidx, val, errno)
- KORE_SYSCALL_DENY_WITH_FLAG(name, argidx, val, errno)

- KORE_SYSCALL_ALLOW(name)
- KORE_SYSCALL_ALLOW_LOG(name)
- KORE_SYSCALL_ALLOW_ARG(name, argidx, val)
- KORE_SYSCALL_ALLOW_MASK(name, argidx, val)
- KORE_SYSCALL_ALLOW_WITH_FLAG(name, argidx, val)
