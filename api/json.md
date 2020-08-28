# JSON parser

Kore provides a built-in JSON parser for your applications.

## Index

* [kore\_json\_init](#init)
* [kore\_json\_cleanup](#cleanup)
* [kore\_json\_parse](#parse)
* [kore\_json\_find\_object](#findobject)
* [kore\_json\_find\_array](#findarray)
* [kore\_json\_find\_string](#findstring)
* [kore\_json\_find\_number](#findnumber)
* [kore\_json\_find\_literal](#findliteral)

* [kore\_json\_create\_object](#createobject)
* [kore\_json\_create\_array](#createarray)
* [kore\_json\_create\_string](#createstring)
* [kore\_json\_create\_number](#createnumber)
* [kore\_json\_create\_literal](#createliteral)

* [kore\_json\_strerror](#strerror)

## Examples

See the included [example](https://github.com/jorisvink/kore/tree/master/examples/json) in the Kore source tree for an example on how to use this API.

---

# kore\_json\_init {#init}
### Synopsis
```
void kore_json_init(struct kore_json *json, const u_int8_t *data, size_t len);
```
### Description
Initializes a **kore\_json** context. This must be called before any other
function can be safely used.

| Parameter | Description |
| -- | -- |
| json | A kore\_json data structure. |
| data | The JSON data to be parsed. |
| len | The length of the JSON data to be parsed. |

### Returns
Nothing

---
# kore\_json\_cleanup {#cleanup}
### Synopsis
```
void kore_json_cleanup(struct kore_json *json);
```
### Description
Cleanup the **kore\_json** context and release all associated resources.
This must be called when you no longer need the context.

| Parameter | Description |
| -- | -- |
| json | A kore\_json data structure. |

### Returns
Nothing

---
# kore\_json\_parse {#parse}
### Synopsis
```
int kore_json_parse(struct kore_json *json);
```
### Description
Parse the JSON data that was set via **kore\_json\_init**.

| Parameter | Description |
| -- | -- |
| json | A kore\_json data structure. |

### Returns
Returns KORE\_RESULT\_OK if parsing was successfull or KORE\_RESULT\_ERROR
if the parsing failed. If the parsing failed the json-\>error field holds
the error code.

This error code can be translated to a human readable string via the
kore\_json\_strerror() function.

---
# kore\_json\_find\_object {#findobject}
### Synopsis
```
struct kore_json_item *kore_json_find_object(struct kore_json_item *item, const char *name);
```
### Description
Find an object named **name** in the JSON root **item**.

| Parameter | Description |
| -- | -- |
| item | A kore\_json\_item data structure representing the root. |
| name | The name of the object to be found.|

### Returns
A pointer to the JSON object as a kore\_json\_item data structure or NULL
if the object was not found.

```c
struct kore_json_item		*object, *item;

if ((object = kore_json_find_object(json.root, "person")) == NULL)
	return;

TAILQ_FOREACH(item, &object->data.items, list) {
	printf("got a entry in the object of type %d\n", item->type);
}
```

---
# kore\_json\_find\_array {#findarray}
### Synopsis
```
struct kore_json_item *kore_json_find_array(struct kore_json_item *item, const char *name);
```
### Description
Find an array named **name** in the JSON root **item**.

| Parameter | Description |
| -- | -- |
| item | A kore\_json\_item data structure representing the root. |
| name | The name of the array to be found.|

### Returns
A pointer to the JSON array as a kore\_json\_item data structure or NULL
if the array was not found.

```c
struct kore_json_item		*items, *item;

if ((items = kore_json_find_array(json.root, "items")) == NULL)
	return;

TAILQ_FOREACH(item, &items->data.items, list) {
	printf("got a entry in the array of type %d\n", item->type);
}
```

---
# kore\_json\_find\_string {#findstring}
### Synopsis
```
struct kore_json_item *kore_json_find_string(struct kore_json_item *item, const char *name);
```
### Description
Find a string named **name** in the JSON root **item**.

| Parameter | Description |
| -- | -- |
| item | A kore\_json\_item data structure representing the root. |
| name | The name of the string to be found.|

### Returns
A pointer to the JSON string as a kore\_json\_item data structure or NULL
if the string was not found.

### Example

```c
struct kore_json_item		*name;

name = kore_json_find_string(json.root, "name");
if (name != NULL)
	printf("found name '%s'\n", name->data.string);
```

---
# kore\_json\_find\_number {#findnumber}
### Synopsis
```
struct kore_json_item *kore_json_find_number(struct kore_json_item *item, const char *name);
```
### Description
Find a number named **name** in the JSON root **item**.

| Parameter | Description |
| -- | -- |
| item | A kore\_json\_item data structure representing the root. |
| name | The name of the number to be found.|

### Returns
A pointer to the JSON number as a kore\_json\_item data structure or NULL
if the number was not found.

### Example

```c
struct kore_json_item		*price;

price = kore_json_find_number(json.root, "price");
if (price != NULL)
	printf("found 'price' with value of %.0f\n", price->data.number);
```

---
# kore\_json\_find\_literal {#findliteral}
### Synopsis
```
struct kore_json_item *kore_json_find_literal(struct kore_json_item *item, const char *name);
```
### Description
Find a literal named **name** in the JSON root **item**.

| Parameter | Description |
| -- | -- |
| item | A kore\_json\_item data structure representing the root. |
| name | The name of the literal to be found.|

### Returns
A pointer to the JSON literal as a kore\_json\_item data structure or NULL
if the literal was not found.

### Possible values

KORE\_JSON\_TRUE		- The JSON true value
KORE\_JSON\_FALSE		- The JSON false value
KORE\_JSON\_NULL		- The JSON null value

### Example

```c
struct kore_json_item		*istrue;

istrue = kore_json_find_literal(json.root, "istrue");
if (istrue != NULL && istrue->data.literal == KORE_JSON_TRUE)
	printf("istrue is set to true\n");
```

---
# kore\_json\_create\_object {#createobject}
### Synopsis
```
struct kore_json_item *kore_json_create_object(struct kore_json_item *parent, const char *name);
```
### Description
Creates a new JSON object under the given **parent** with the given **name**.

| Parameter | Description |
| -- | -- |
| parent | A kore\_json\_item data structure representing the parent. |
| name | The name of the object to be created.|

### Parent
The parent parameter can be NULL if no parent is to be used.

### Returns
A pointer to the JSON object as a kore\_json\_item data structure or NULL
if something has failed.

### Example

```c
struct kore_json_item		*person;

if ((person = kore_json_create_object(NULL, "person")) == NULL)
	/* handle */

if (kore_json_create_number(person, "age", 34) == NULL)
	/* handle */
```

---
# kore\_json\_create\_array {#createarray}
### Synopsis
```
struct kore_json_item *kore_json_create_array(struct kore_json_item *parent, const char *name);
```
### Description
Creates a new JSON array under the given **parent** with the given **name**.

| Parameter | Description |
| -- | -- |
| parent | A kore\_json\_item data structure representing the parent. |
| name | The name of the array to be created.|

### Parent
The parent parameter can be NULL if no parent is to be used.

### Returns
A pointer to the JSON array as a kore\_json\_item data structure or NULL
if something has failed.

### Example

```c
struct kore_json_item		*array;

if ((array = kore_json_create_array(NULL, "person")) == NULL)
	/* handle */

if (kore_json_create_number(array, NULL, 34) == NULL)
	/* handle */

if (kore_json_create_number(array, NULL, 27) == NULL)
	/* handle */
```

---
# kore\_json\_create\_string {#createstring}
### Synopsis
```
struct kore_json_item *kore_json_create_string(struct kore_json_item *parent, const char *name, const char *value);
```
### Description
Creates a new JSON string under the given **parent** with the given **name**.

| Parameter | Description |
| -- | -- |
| parent | A kore\_json\_item data structure representing the parent. |
| name | The name of the string to be created.|
| value | The value of the string. |

### Parent
The parent parameter can be NULL if no parent is to be used.

### Returns
A pointer to the JSON string as a kore\_json\_item data structure or NULL
if something has failed.

### Example

```c
if (kore_json_create_string(parent, "person", "hacker") == NULL)
	/* handle */
```

---
# kore\_json\_create\_number {#createnumber}
### Synopsis
```
struct kore_json_item *kore_json_create_number(struct kore_json_item *parent, const char *name, double value);
```
### Description
Creates a new JSON number under the given **parent** with the given **name**.

| Parameter | Description |
| -- | -- |
| parent | A kore\_json\_item data structure representing the parent. |
| name | The name of the number to be created.|
| value | The value of the number as a C double. |

### Parent
The parent parameter can be NULL if no parent is to be used.

### Returns
A pointer to the JSON number as a kore\_json\_item data structure or NULL
if something has failed.

### Example

```c
if (kore_json_create_number(parent, "age", 34) == NULL)
	/* handle */
```

---
# kore\_json\_create\_literal {#createliteral}
### Synopsis
```
struct kore_json_item *kore_json_create_literal(struct kore_json_item *parent, const char *name, int value);
```
### Description
Creates a new JSON literal under the given **parent** with the given **name**.

| Parameter | Description |
| -- | -- |
| parent | A kore\_json\_item data structure representing the parent. |
| name | The name of the literal to be created.|
| value | The value of the literal (see below). |

### Parent
The parent parameter can be NULL if no parent is to be used.

### Possible values

KORE\_JSON\_TRUE		- The JSON true value
KORE\_JSON\_FALSE		- The JSON false value
KORE\_JSON\_NULL		- The JSON null value

### Returns
A pointer to the JSON literal as a kore\_json\_item data structure or NULL
if something has failed.

### Example

```c
if (kore_json_create_literal(parent, "istrue", KORE_JSON_FALSE) == NULL)
	/* handle */
```
