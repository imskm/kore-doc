# Pools

Kore provides a memory pool interface allowing you to create your own.

Pools are automatically thread-safe if Kore was built with TASKS=1.

---

# kore_pool_init
### Synopsis
```
void kore_pool_init(struct kore_pool *pool, const char *name, size_t len, size_t elm)
```
### Description
Initializes a new pool.

| Parameter | Description |
| -- | -- |
| pool | A pointer to a pool. |
| name | The name to give to the pool. This name is shown in the logs if the pool is exhausted. |
| len | The size in bytes of each element. |
| elm | The number of initial elements to preallocate to the pool. |

### Returns
Nothing

---

# kore_pool_cleanup
### Synopsis
```
void kore_pool_cleanup(struct kore_pool *pool)
```
### Description
Deallocate and cleanup a pool.

| Parameter | Description |
| -- | -- |
| pool | A pointer to a pool. |

### Returns
Nothing

---

# kore_pool_get
### Synopsis
```
void *kore_pool_get(struct kore_pool *pool)
```
### Description
Obtain a pointer to a free element from the given pool. If a pool runs out of free elements it is automatically grown in size.

| Parameter | Description |
| -- | -- |
| pool | A pointer to a pool. |

### Returns
Returns a pointer to an area that is large enough to hold the data length the pool was initializes with.

---

# kore_pool_put
### Synopsis
```
void kore_pool_put(struct kore_pool *pool, void *ptr)
```
### Description
Returns the given pointer back to its pool.

| Parameter | Description |
| -- | -- |
| pool | A pointer to a pool. |
| ptr | The pointer to be returned to the pool. |

### Returns
Nothing

---