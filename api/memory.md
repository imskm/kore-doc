# Memory

The memory system in Kore for heap allocation is based on pools. At startup Kore will initialize at least 10 pools for commonly sized objects \(ranging from 8 to 8192 bytes\).

Any data allocated via _kore\_malloc\(\)_, _kore\_strdup\(\)_ or _kore\_realloc\(\)_ comes from the common pools unless it is larger then 8192 in which case _calloc\(\)_ is used.

---

# kore\_malloc

### Synopsis

```
void *kore_malloc(size_t length)
```

### Description

Allocates new data.

| Parameter | Description |
| --- | --- |
| length | The length of the data to allocate. |

### Returns

A pointer to allocated memory that can hold _length_ bytes. The returned pointer is aligned for use with any data type.

---

# kore\_calloc

### Synopsis

```
void *kore_calloc(size_t memb, size_t len)
```

### Description

Allocates new data and zeroes it out.

| Parameter | Description |
| --- | --- |
| memb | Number of objects. |
| len | Size of each object. |

### Returns

A pointer to allocated memory that can hold _ memb \* len_ bytes. The returned pointer is aligned for use with any data type and the contents is zeroed out.

---

# kore\_realloc

### Synopsis

```
void *kore_realloc(void *ptr, size_t length)
```

### Description

Grows or shrinks a existing allocated memory pointer.

| Parameter | Description |
| --- | --- |
| ptr | The pointer to reallocate. |
| length | The new length of the data. |

### Returns

A pointer to the newly allocated memory.

---

# kore\_free

### Synopsis

```
void kore_free(void *ptr)
```

### Description

Frees a previously allocated pointer.

| Parameter | Description |
| --- | --- |
| ptr | The pointer to be freed. |

### Returns

Nothing

---

# kore\_strdup

### Synopsis

```
char *kore_strdup(const char *str)
```

### Description

Duplicate an existing C string.

| Parameter | Description |
| --- | --- |
| str | The C string to be duplicated. |

### Returns

A pointer to the duplicated C string.

---



