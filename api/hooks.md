# Hooks

Kore supports several hooking functions allowing your application
to do things at certain times in the Kore startup / shutdown procedure.

* [kore\_parent\_configure](#parentconfigure)
* [kore\_parent\_daemonized](#parentdaemonized)
* [kore\_parent\_teardown](#parentteardown)
* [kore\_worker\_configure](#workerconfigure)
* [kore\_worker\_teardown](#workerteardown)
* [kore\_python\_preinit](#pythonpreinit)

---

# kore\_parent\_configure {#parentconfigure}

### Synopsis

```
void kore_parent_configure(int argc, char *argv[])
```

### Description

Called immediately after Kore is done parsing the command line arguments
and before any configuration is read.

This is called only once.

| Parameter | Description |
| --- | --- |
| argc | The number of remaining command line arguments. |
| argv | The remaining command line arguments. |

### Returns

Nothing

---

# kore\_parent\_daemonized {#parentdaemonized}

### Synopsis

```
void kore_parent_daemonized(void)
```

### Description

Called after the parent process has daemonized itself.

This is called only once.

### Returns

Nothing

---

# kore\_parent\_teardown {#parentteardown}

### Synopsis

```
void kore_parent_teardown(void)
```

### Description

Called right before the parent process will exit.

This is called only once.

### Returns

Nothing

---

# kore\_worker\_configure {#workerconfigure}

### Synopsis

```
void kore_worker_configure(void)
```

### Description

Called by a worker process immediately after it is done initializing itself.

(This is called **per** worker).

### Returns

Nothing

---

# kore\_worker\_teardown {#workerteardown}

### Synopsis

```
void kore_worker_teardown(void)
```

### Description

Called by a worker process right before it exits.

(This is called **per** worker).

### Returns

Nothing

---

# kore\_python\_preinit {#pythonpreinit}

### Synopsis

```
void kore_python_preinit(void)
```

### Description

If Python support is enabled Kore will call this hook right before
it calls Py\_Initialize().

This allows your application to register its own built-in modules
that can be used by your Python code.

### Returns

Nothing
