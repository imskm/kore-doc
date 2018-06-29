# Buffers

The kore_buf interface provides a safe way of constructing data. Buffers automatically grow when more data is written to them and can either exist on the heap or on the stack.

---

# kore_buf_alloc
### Synopsis
```
struct kore_buf *kore_buf_alloc(size_t initial)
```
### Description
Allocates a new *kore_buf* data structure on the heap and initializes it.

| Parameter | Description |
| -- | -- |
| initial | The number of initial bytes to allocate for the buffer data. Can be 0. |

### Returns
A pointer to a new *kore_buf* data structure. This must be freed by the caller using *kore_buf_free()*.

---

# kore_buf_init
### Synopsis
```
void kore_buf_init(struct kore_buf *buf, size_t initial)
```
### Description
Initializes the given buffer.

| Parameter | Description |
| -- | -- |
| buf | The buffer to initialize. |
| initial | The number of initial bytes to allocate for the buffer data. Can be 0. |

### Returns
Nothing.

---

# kore_buf_cleanup
### Synopsis
```
void kore_buf_cleanup(struct kore_buf *buf)
```
### Description
Cleanup the given buffer by releasing the buffer data and resetting its members.

| Parameter | Description |
| -- | -- |
| buf | The buffer to be cleaned up. |

### Returns
Nothing

---

# kore_buf_free
### Synopsis
```
void kore_buf_free(struct kore_buf *buf)
```
### Description
Cleanup and free a previously allocated buffer.

| Parameter | Description |
| -- | -- |
| buf | The buffer to be cleaned up and freed. |

### Returns
Nothing

---

# kore_buf_append
### Synopsis
```
void kore_buf_append(struct kore_buf *buf, const void *data, size_t length)
```
### Description
Append some data to a buffer. This function will automatically grow the buffer data if required before appending the given data.

| Parameter | Description |
| -- | -- |
| buf | The buffer. |
| data | The data to be appended. |
| length | The length of the data to be appended .|

### Returns
Nothing

---

# kore_buf_appendv
### Synopsis
```
void kore_buf_appendv(struct kore_buf *buf, const char *fmt, va_list args)
```
### Description
Append data given in the form of a format and a variadic argument list.

| Parameter | Description |
| -- | -- |
| buf | The buffer. |
| fmt | The format string. |
| args | Variadic argument list. |


### Returns
Nothing

---

# kore_buf_appendf
### Synopsis
```
void kore_buf_appendf(struct kore_buf *buf, const char *fmt, ...)
```
### Description
Append data given in the form of a string format and list of arguments. Much like snprintf().

| Parameter | Description |
| -- | -- |
| buf | The buffer. |
| fmt | The format string. |
| ... | Any arguments for the format string. |

### Returns
Nothing

---

# kore_buf_stringify
### Synopsis
```
char *kore_buf_stringify(struct kore_buf *buf, size_t *length)
```
### Description
Returns the data in a buffer as a C string. The result is NUL-terminated before it is returning to the caller. The caller should not free this data.

| Parameter | Description |
| -- | -- |
| buf | The buffer. |
| length | optional, if not NULL the length of the C string is placed here. |

### Returns
A pointer to the C string.

---

# kore_buf_release
### Synopsis
```
u_int8_t *kore_buf_release(struct kore_buf *buf, size_t *length)
```
### Description
Release the buffer data from a buffer. This causes the buffer to be cleaned up.

| Parameter | Description |
| -- | -- |
| buf | The buffer. |
| length | The length of the returned data is placed here. |

### Returns
A pointer to the original buffer data.

---

