# Tasks

The tasks API in Kore allows you to create background tasks which are scheduled over OS threads.

Background tasks can communicate with the main thread via a socket pair.

Kore must be built with TASKS=1 in order to use this API.

---

# kore_task_create
### Synopsis
```
void kore_task_create(struct kore_task *t, int (*entry)(struct kore_task *))
```
### Description
Creates a new task which when run will call the given callback.

| Parameter | Description |
| -- | -- |
| t | A task data structure. |
| entry | The entry point of the task when run. |

### Returns
Nothing

---
# kore_task_run
### Synopsis
```
void kore_task_run(struct kore_task *t)
```
### Description
Allows a task to be run. After this function returns the task will be scheduled to run.

| Parameter | Description |
| -- | -- |
| t | A task data structure. |

### Returns
Nothing

---
# kore_task_bind_request
### Synopsis
```
void kore_task_bind_request(struct kore_task *t, struct http_request *req)
```
### Description
Bind an HTTP request to a task. Binding means that the HTTP request will be scheduled to be called again by Kore if the task writes data on the task channel or when it completes.

Using this in combination with the HTTP state machine allows you to build request handlers that use background tasks for heavy labor.

| Parameter | Description |
| -- | -- |
| t | A task data structure. |
| req | The HTTP request to be bound to the task. |

### Returns
Nothing

---
# kore_task_bind_callback
### Synopsis
```
void kore_task_bind_callback(struct kore_task *t, void (*cb)(struct kore_task *))
```
### Description
Bind a callback to a task. Binding means that the callback will be called whenever the task writes data on the task channel or whenever it completes.

| Parameter | Description |
| -- | -- |
| t | A task data structure. |
| cb | The callback to call. |

### Returns
Nothing

---
# kore_task_destroy
### Synopsis
```
void kore_task_destroy(struct kore_task *t)
```
### Description
Destroys a task.

| Parameter | Description |
| -- | -- |
| t | A task data structure. |

### Returns
Nothing

---
# kore_task_finished
### Synopsis
```
int kore_task_finished(struct kore_task *t)
```
### Description
Check if a task has finished running.

| Parameter | Description |
| -- | -- |
| t | A task data structure. |

### Returns
Returns 1 if the task has finished running, otherwise 0.

---
# kore_task_channel_write
### Synopsis
```
void kore_task_channel_write(struct kore_task *t, void *data, u_int32_t length)
```
### Description
Write data on the task channel for the other end to read. This works both from the main thread and a task itself.

| Parameter | Description |
| -- | -- |
| t | A task data structure. |
| data | The data to be written on the channel. |
| length | The length of the data to be written. |

### Returns
Nothing

---

# kore_task_channel_read
### Synopsis
```
u_int32_t kore_task_channel_read(struct kore_task *t, void *out, u_int32_t length)
```
### Description
Read data from the task channel.

NOTE: This is a blocking operation.

| Parameter | Description |
| -- | -- |
| t | A task data structure. |
| out | Where the data read will be written too. |
| length | The maximum number of bytes that *out* can hold. |

### Returns
Returns the number of original bytes from the message, if this is larger then
the *length* parameter then truncation has occurred.

---