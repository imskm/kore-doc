# Pgsql

The pgsql API in Kore allows you to use PostgreSQL in a straight forward manner. It supports both synchronous and asynchronous queries.

Note that you must have built Kore with PGSQL=1 in order to use this API.

---

# kore\_pgsql\_init

### Synopsis

```
void kore_pgsql_init(struct kore_pgsql *pgsql)
```

### Description

Initialize a kore\_pgsql data structure for use.

| Parameter | Description |
| --- | --- |
| pgsql | A pgsql data structure. |

### Returns

Nothing.

---

# kore\_pgsql\_setup

### Synopsis

```
int kore_pgsql_query_init(struct kore_pgsql *pgsql, const char *dbname, int flags)
```

### Description

Setup a pgsql data structure for use.

| Parameter | Description |
| --- | --- |
| pgsql | A pgsql data structure. |
| dbname | The name of the database connection to be used. |
| flags | Any additional flags. |

| Flag | Meaning |
| --- | --- |
| KORE\_PGSQL\_ASYNC | Setup an asynchronous query. |
| KORE\_PGSQL\_SYNC | Setup a synchronous query. |

### Returns

KORE\_RESULT\_OK or KORE\_RESULT\_ERROR. If KORE\_RESULT\_ERROR was returned but the _pgsql-&gt;state_ is still set to _KORE\_PGSQL\_STATE\_INIT_ then you can try again.

---

# kore\_pgsql\_bind\_request

### Synopsis

```
void kore_pgsql_bind_request(struct kore_pgsql *pgsql, struct http_request *req)
```

### Description

Bind a kore\_pgsql data structure to an HTTP request. This causes the HTTP request to be notified for any changes to the pgsql data structure.

| Parameter | Description |
| --- | --- |
| pgsql | A pgsql data structure. |
| req | The HTTP request to be tied to. |

### Returns

Nothing.

---

# kore\_pgsql\_bind\_callback

### Synopsis

```
void kore_pgsql_bind_callback(struct kore_pgsql *pgsql, void (*cb)(struct kore_pgsql *, void *), void *ar)
```

### Description

Bind a kore\_pgsql data structure to a callback function. This function is called for any state change related to the pgsql data structure.

| Parameter | Description |
| --- | --- |
| pgsql | A pgsql data structure. |
| cb | Pointer to a C function. |

### Returns

Nothing.

---

# kore\_pgsql\_query

### Synopsis

```
int kore_pgsql_query(struct kore_pgsql *pgsql, const char *query)
```

### Description

Fires of a query.

| Parameter | Description |
| --- | --- |
| pgsql | A previously initialized pgsql data structure. |
| query | The query to be sent. |

### Returns

KORE\_RESULT\_OK or KORE\_RESULT\_ERROR.

---

# kore\_pgsql\_query\_params

### Synopsis

```
int kore_pgsql_query_params(struct kore_pgsql *pgsql, const char *query, int result, int count, ...)
```

### Description

Creates and runs a parameterized query

| Parameter | Description |
| --- | --- |
| pgsql | A previously initialized pgsql data structure. |
| query | The query to send, has `$1`,`$2`,... in places where values are substituted |
| result | 0 if the query's result should be a string, 1 if the query's result should be binary data. |
| count | The number of parameters to substitute in a query |
| ... | The parameters to substitute into the query. Parameters are passed in groups of 3 :`value`, `length`, `format`. |

The `format` argument may be 0 (`value` is a null-terminated string) or 1 (`value` is binary data of length `length` binary). If `format` is 0, `length` may be set to 0 to have the system find the length of `value`.

### Returns

KORE\_RESULT\_OK or KORE\_RESULT\_ERROR.

---

# kore\_pgsql\_register

### Synopsis

```
int kore_pgsql_register(const char *dbname, const char *connstring)
```

### Description

Register a database connection.

| Parameter | Description |
| --- | --- |
| dbname | The name to give to this connection. |
| connstring | A PostgreSQL connection string. |

### Returns

KORE\_RESULT\_OK or KORE\_RESULT\_ERROR if the dbname was already taken.

---

# kore\_pgsql\_continue

### Synopsis

```
void kore_pgsql_continue(struct http_request *req, struct kore_pgsql *pgsql)
```

### Description

Continue the asynchronous state machine. Usually called from HTTP state machines or callback functions.

| Parameter | Description |
| --- | --- |
| req | The HTTP request we are dealing with. |
| pgsql | The pgsql we are dealing with. |

### Returns

Nothing

---

# kore\_pgsql\_cleanup

### Synopsis

```
void kore_pgsql_cleanup(struct kore_pgsql *pgsql)
```

### Description

Cleanup a previously initialized pgsql data structure and relinquish the database connection it held.

| Parameter | Description |
| --- | --- |
|  |  |

### Returns

Nothing

---

# kore\_pgsql\_logerror

### Synopsis

```
void kore_pgsql_logerror(struct kore_pgsql *pgsql)
```

### Description

Log what error occurred to syslog with a LOG\_NOTICE priority.

| Parameter | Description |
| --- | --- |
| pgsql | The pgsql data structure. |

### Returns

Nothing

---

# kore\_pgsql\_ntuples

### Synopsis

```
int kore_pgsql_ntuples(struct kore_pgsql *pgsql)
```

### Description

Returns the number of tuples affected by a query.

| Parameter | Description |
| --- | --- |
| pgsql | The pgsql data structure. |

### Returns

The number of tuples.

---

# kore\_pgsql\_nfields

### Synopsis

```
int kore_pgsql_nfields(struct kore_pgsql *pgsql)
```

### Description

Returns the number of fields in the result.

| Parameter | Description |
| --- | --- |
| pgsql | The pgsql data structure. |

### Returns

The number of fields.

---

# kore\_pgsql\_getlength

### Synopsis

```
int kore_pgsql_getlength(struct kore_pgsql *pgsql, int row, int col)
```

### Description

Returns the length of the data at row, col.

| Parameter | Description |
| --- | --- |
| pgsql | The pgsql data structure. |
| row | The row number. |
| col | The column number. |

### Returns

The length of the data at row, col.

---

# kore\_pgsql\_getvalue

### Synopsis

```
char *kore_pgsql_getvalue(struct kore_pgsql *pgsql, int row, int col)
```

### Description

Returns the value of the data at row, col.

| Parameter | Description |
| --- | --- |
| pgsql | The pgsql data structure. |
| row | The row number. |
| col | The column number. |

### Returns

A pointer to the data. This should not be freed by the caller.

---

# kore\_pgsql\_column\_binary

### Synopsis

```
int kore_pgsql_column_binary(struct kore_pgsql *pgsql, int col)
```

### Description

Test if the given column contains binary or text data.

| Parameter | Description |
| --- | --- |
| pgsql | The pgsql data structure. |
| col | The column number. |

### Returns

Returns 1 if the column contains binary data or 0 if text.

---
