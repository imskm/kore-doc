# Miscellaneous

Kore provides several helper functions.

---

# kore_log
### Synopsis
```
void kore_log(int prio, const char *fmt, ...)
```
### Description
Output a log message via syslog.

Note that when running in the foreground these go to stdout.

| Parameter | Description |
| -- | -- |
| prio   |  The priority of the log message (see syslog(3)).  |
| fmt | The format string. |
| ... | List of arguments. |

### Returns
Nothing

---

# kore_strlcpy
### Synopsis
```
size_t kore_strlcpy(char *dst, const char *src, const size_t length)
```
### Description
Bounded C string copying. The result is always NUL-terminated.

| Parameter | Description |
| -- | -- |
| dst | The destination buffer. |
| src | The source C string. |
| length | The maximum number of bytes that the destination buffer holds. |

### Returns
The length of the original string. If this length is equal or larger then the destination buffer then truncation of the string has occurred.

---

# kore_strtonum
### Synopsis
```
long long kore_strtonum(const char *str, int base, long long min, long long max, int *err)
```
### Description
Safely convert a C string holding a number into an integer.

| Parameter | Description |
| -- | -- |
| str | The C string to convert. |
| base | The base on which to operate. |
| min | The minimum value the converted integer is allowed to have. |
| max | The maximum value the converted integer is allowed to have. |
| err | A pointer to an integer that holds the error value. |

### Returns
Returns the converted integer. If the *err* parameter was set to KORE_RESULT_OK the conversion went OK otherwise there was an error and the returned value is considered garbage.

---

# kore_strtonum64
### Synopsis
```
u_int64_t kore_strtonum64(const char *str, int sign, int *err)
```
### Description
Safely convert a C string holding a number to a 64-bit integer.

| Parameter | Description |
| -- | -- |
| str | The C string to convert. |
| sign | If the 64-bit integer is allowed to be signed or not. |
| err | A pointer to an integer that holds the error value. |

### Returns
Returns the converted integer. If the *err* parameter was set to KORE_RESULT_OK the conversion went OK otherwise there was an error and the returned value is considered garbage.

---

# kore_split_string
### Synopsis
```
int kore_split_string(char *input, char *delim, char **out, size_t ele)
```
### Description
Safely split a string into several elements. The *out* array is NULL terminated.

| Parameter | Description |
| -- | -- |
| input | The string to be split up. |
| delim | The delimiter used for splitting. |
| out | Pointer to an array where the components are stored. |
| ele | The number of elements that can be held in *out*. |

### Returns
The number of components.

---

# kore_strip_chars
### Synopsis
```
void kore_strip_chars(char *in, const char strip, char **out)
```
### Description
Safely strip all occurences of a certain character from a C string. The result must be freed by the caller.

| Parameter | Description |
| -- | -- |
| in | The input C string. |
| strip | The character to be stripped out of the string. |
| out | A pointer to where the new result is stored. |

### Returns
Nothing

---