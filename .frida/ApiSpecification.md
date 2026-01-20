# API Specification

##### POST /tasks
**Description:** Create a new task with the specified details. The task will be assigned to the default "Uncategorized" category if no category is specified. Creation and update timestamps are set automatically.

**Authentication:** Not Required

**Request Body Schema:**
| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| title | string | Yes | Min 1 char, Max 200 chars, Not whitespace only | The name or title of the task |
| description | text | No | Max 2000 chars | Detailed description or notes about the task |
| dueDate | date | No | Valid date format (ISO 8601) | The date by which the task should be completed |
| priority | enum | Yes | Values: Low, Medium, High | The importance level of the task |
| categoryId | identifier | No | Must reference existing category | The category this task belongs to |

**Response Schema:**
| Field | Type | Description |
|-------|------|-------------|
| id | identifier | Unique identifier for the created task |
| title | string | The name or title of the task |
| description | text | Detailed description or notes about the task |
| dueDate | date | The date by which the task should be completed |
| priority | enum | The importance level (Low, Medium, High) |
| isComplete | boolean | Completion status (always false on creation) |
| categoryId | identifier | The category this task belongs to |
| createdAt | datetime | Timestamp when the task was created |
| updatedAt | datetime | Timestamp when the task was last modified |

**Business Rules:**
- Title cannot be empty or contain only whitespace
- Priority is required and must be one of: Low, Medium, High
- isComplete is automatically set to false on creation
- createdAt and updatedAt are automatically set to current datetime
- If categoryId is not provided, task is assigned to the default "Uncategorized" category
- If categoryId is provided, it must reference an existing category

**Error Responses:**
- 400 if title is missing, empty, or exceeds 200 characters
- 400 if title contains only whitespace
- 400 if priority is missing or not a valid enum value
- 400 if description exceeds 2000 characters
- 400 if dueDate is not a valid date format
- 400 if categoryId does not reference an existing category

---

##### GET /tasks
**Description:** Retrieve all tasks with support for filtering by completion status, priority, and due date range. Supports sorting by due date, priority, creation date, or title. Supports searching across task titles and descriptions. Results are paginated.

**Authentication:** Not Required

**Query Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| page | integer | No | 1 | Page number for pagination (min: 1) |
| limit | integer | No | 20 | Number of items per page (min: 1, max: 100) |
| status | enum | No | all | Filter by completion status. Values: all, active, completed |
| priority | enum | No | - | Filter by priority level. Values: Low, Medium, High |
| dueDateFilter | enum | No | - | Filter by due date. Values: overdue, today, upcoming |
| categoryId | identifier | No | - | Filter by category |
| sortBy | enum | No | createdAt | Sort field. Values: dueDate, priority, createdAt, title |
| sortOrder | enum | No | asc | Sort direction. Values: asc, desc |
| search | string | No | - | Search term to match against title and description |

**Response Schema:**
| Field | Type | Description |
|-------|------|-------------|
| data | array | Array of task objects |
| data[].id | identifier | Unique identifier for the task |
| data[].title | string | The name or title of the task |
| data[].description | text | Detailed description or notes about the task |
| data[].dueDate | date | The date by which the task should be completed |
| data[].priority | enum | The importance level (Low, Medium, High) |
| data[].isComplete | boolean | Whether the task has been marked as completed |
| data[].categoryId | identifier | The category this task belongs to |
| data[].createdAt | datetime | Timestamp when the task was created |
| data[].updatedAt | datetime | Timestamp when the task was last modified |
| pagination | object | Pagination metadata |
| pagination.page | integer | Current page number |
| pagination.limit | integer | Items per page |
| pagination.totalItems | integer | Total number of matching tasks |
| pagination.totalPages | integer | Total number of pages |

**Business Rules:**
- Status filter "active" returns tasks where isComplete is false
- Status filter "completed" returns tasks where isComplete is true
- Status filter "all" returns all tasks regardless of completion status
- Due date filter "overdue" returns tasks with dueDate before today and isComplete is false
- Due date filter "today" returns tasks with dueDate equal to today
- Due date filter "upcoming" returns tasks with dueDate after today
- Tasks without a dueDate are excluded when dueDateFilter is applied
- When sorting by priority, order is High > Medium > Low (desc) or Low > Medium > High (asc)
- Search is case-insensitive and matches partial text in title or description
- Multiple filters can be combined (AND logic)
- Empty result set returns empty array with totalItems: 0

**Error Responses:**
- 400 if page is less than 1
- 400 if limit is less than 1 or greater than 100
- 400 if status is not a valid enum value
- 400 if priority is not a valid enum value
- 400 if dueDateFilter is not a valid enum value
- 400 if sortBy is not a valid enum value
- 400 if sortOrder is not a valid enum value
- 400 if categoryId does not reference an existing category

---

##### GET /tasks/{taskId}
**Description:** Retrieve the detailed view of a single task by its unique identifier. Returns all task fields including full description.

**Authentication:** Not Required

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| taskId | identifier | Yes | Unique identifier of the task to retrieve |

**Response Schema:**
| Field | Type | Description |
|-------|------|-------------|
| id | identifier | Unique identifier for the task |
| title | string | The name or title of the task |
| description | text | Detailed description or notes about the task |
| dueDate | date | The date by which the task should be completed |
| priority | enum | The importance level (Low, Medium, High) |
| isComplete | boolean | Whether the task has been marked as completed |
| categoryId | identifier | The category this task belongs to |
| createdAt | datetime | Timestamp when the task was created |
| updatedAt | datetime | Timestamp when the task was last modified |

**Business Rules:**
- Task must exist with the specified taskId
- All task fields are returned in the response

**Error Responses:**
- 404 if task with specified taskId does not exist

---

##### PATCH /tasks/{taskId}
**Description:** Update one or more fields of an existing task. Only the fields provided in the request body will be updated. The updatedAt timestamp is automatically updated on any modification.

**Authentication:** Not Required

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| taskId | identifier | Yes | Unique identifier of the task to update |

**Request Body Schema:**
| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| title | string | No | Min 1 char, Max 200 chars, Not whitespace only | The name or title of the task |
| description | text | No | Max 2000 chars | Detailed description or notes about the task |
| dueDate | date | No | Valid date format (ISO 8601), or null to clear | The date by which the task should be completed |
| priority | enum | No | Values: Low, Medium, High | The importance level of the task |
| categoryId | identifier | No | Must reference existing category | The category this task belongs to |
| isComplete | boolean | No | true or false | Whether the task has been marked as completed |

**Response Schema:**
| Field | Type | Description |
|-------|------|-------------|
| id | identifier | Unique identifier for the task |
| title | string | The name or title of the task |
| description | text | Detailed description or notes about the task |
| dueDate | date | The date by which the task should be completed |
| priority | enum | The importance level (Low, Medium, High) |
| isComplete | boolean | Whether the task has been marked as completed |
| categoryId | identifier | The category this task belongs to |
| createdAt | datetime | Timestamp when the task was created |
| updatedAt | datetime | Timestamp when the task was last modified |

**Business Rules:**
- Task must exist with the specified taskId
- Only fields included in the request body are updated; omitted fields retain their current values
- Title cannot be updated to empty or whitespace only
- updatedAt is automatically set to current datetime when any field is modified
- Setting isComplete to true marks the task as completed
- Setting isComplete to false reverts a completed task to active status
- Task history is preserved (updatedAt timestamp reflects modification)
- dueDate can be set to null to clear the due date

**Error Responses:**
- 404 if task with specified taskId does not exist
- 400 if title is empty or exceeds 200 characters
- 400 if title contains only whitespace
- 400 if priority is not a valid enum value
- 400 if description exceeds 2000 characters
- 400 if dueDate is not a valid date format
- 400 if categoryId does not reference an existing category
- 400 if request body is empty (no fields to update)

---

##### DELETE /tasks/{taskId}
**Description:** Permanently delete a task. This action cannot be undone. The client should request user confirmation before calling this endpoint.

**Authentication:** Not Required

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| taskId | identifier | Yes | Unique identifier of the task to delete |

**Response Schema:**
| Field | Type | Description |
|-------|------|-------------|
| message | string | Confirmation message indicating successful deletion |
| deletedId | identifier | The identifier of the deleted task |

**Business Rules:**
- Task must exist with the specified taskId
- Deletion is permanent; there is no soft delete or recycle bin
- Completed tasks can be deleted
- Active tasks can be deleted
- Client application should display confirmation dialog before calling this endpoint (FR-4.2)

**Error Responses:**
- 404 if task with specified taskId does not exist

##### POST /categories
**Description:** Create a new custom category for organizing tasks. Categories allow users to group related tasks together for better organization. The category name must be unique across all user categories.

**Authentication:** Required

**Request Body Schema:**
| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| name | string | Yes | Min 1 char, Max 100 chars, Unique | The display name for the category |

**Response Schema:**
| Field | Type | Description |
|-------|------|-------------|
| id | identifier | Unique identifier for the created category |
| name | string | The display name of the category |
| isDefault | boolean | Always false for user-created categories |
| createdAt | datetime | Timestamp when the category was created |

**Business Rules:**
- Category name cannot be empty or contain only whitespace
- Category name must be unique (case-insensitive comparison recommended)
- isDefault is automatically set to false for all user-created categories
- createdAt is automatically set to current datetime on creation
- Leading and trailing whitespace should be trimmed from the name

**Error Responses:**
- 400 if name is missing, empty, or contains only whitespace
- 400 if name exceeds maximum length
- 409 if a category with the same name already exists
- 401 if not authenticated

---

##### GET /categories
**Description:** Retrieve all categories including the system default "Uncategorized" category. Returns the complete list of categories available for organizing tasks. The default category is always included and typically displayed first.

**Authentication:** Required

**Query Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| includeTaskCount | boolean | No | false | Include count of tasks in each category |

**Response Schema:**
| Field | Type | Description |
|-------|------|-------------|
| categories | array | List of category objects |
| categories[].id | identifier | Unique identifier for the category |
| categories[].name | string | The display name of the category |
| categories[].isDefault | boolean | True if this is the system default category |
| categories[].createdAt | datetime | Timestamp when the category was created |
| categories[].taskCount | integer | Number of tasks in category (only if includeTaskCount=true) |

**Business Rules:**
- The default "Uncategorized" category is always included in the response
- Categories should be sorted with the default category first, then alphabetically by name
- If includeTaskCount is true, include the count of all tasks (both complete and incomplete) in each category

**Error Responses:**
- 401 if not authenticated

---

##### PATCH /categories/{categoryId}
**Description:** Rename an existing category. Allows users to update the display name of their custom categories. The default "Uncategorized" category cannot be renamed.

**Authentication:** Required

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| categoryId | identifier | Yes | Unique identifier of the category to rename |

**Request Body Schema:**
| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| name | string | Yes | Min 1 char, Max 100 chars, Unique | The new display name for the category |

**Response Schema:**
| Field | Type | Description |
|-------|------|-------------|
| id | identifier | Unique identifier for the category |
| name | string | The updated display name of the category |
| isDefault | boolean | Whether this is the default category (always false for renameable categories) |
| createdAt | datetime | Timestamp when the category was created |

**Business Rules:**
- The default category (isDefault: true) cannot be renamed
- New category name cannot be empty or contain only whitespace
- New category name must be unique across all categories
- Leading and trailing whitespace should be trimmed from the name
- Renaming to the same name (no change) should succeed without error

**Error Responses:**
- 400 if name is missing, empty, or contains only whitespace
- 400 if name exceeds maximum length
- 403 if attempting to rename the default category
- 404 if category with specified categoryId does not exist
- 409 if a different category with the same name already exists
- 401 if not authenticated

---

##### DELETE /categories/{categoryId}
**Description:** Delete a custom category. When a category is deleted, all tasks assigned to it are automatically moved to the default "Uncategorized" category to maintain data integrity. The default category cannot be deleted.

**Authentication:** Required

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| categoryId | identifier | Yes | Unique identifier of the category to delete |

**Response Schema:**
| Field | Type | Description |
|-------|------|-------------|
| message | string | Confirmation message indicating successful deletion |
| tasksReassigned | integer | Number of tasks that were moved to the default category |

**Business Rules:**
- The default category (isDefault: true) cannot be deleted
- All tasks belonging to the deleted category must be reassigned to the default "Uncategorized" category before deletion
- The deletion is permanent and cannot be undone
- The response should indicate how many tasks were reassigned to inform the user of the impact

**Error Responses:**
- 403 if attempting to delete the default category
- 404 if category with specified categoryId does not exist
- 401 if not authenticated

