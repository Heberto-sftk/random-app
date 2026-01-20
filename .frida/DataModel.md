# Entities
### Task
**Description:** Represents an individual task or to-do item that a user needs to track and complete

**Attributes:**
| Attribute | Type | Required | Constraints | Description |
|-----------|------|----------|-------------|-------------|
| id | identifier | Yes | Primary Key, Unique | Unique identifier for the task |
| title | string | Yes | Min 1 char, Max 200 chars | The name or title of the task |
| description | text | No | Max 2000 chars | Detailed description or notes about the task |
| dueDate | date | No | Valid date format | The date by which the task should be completed |
| priority | enum | Yes | Values: Low, Medium, High | The importance level of the task |
| isComplete | boolean | Yes | Default: false | Whether the task has been marked as completed |
| categoryId | identifier | No | Foreign Key → Category | The category this task belongs to |
| createdAt | datetime | Yes | Auto-generated on creation | Timestamp when the task was created |
| updatedAt | datetime | Yes | Auto-updated on modification | Timestamp when the task was last modified |

**Business Rules:**
- Title cannot be empty or whitespace only
- Priority must be one of: Low, Medium, High
- createdAt is automatically set to current datetime when task is created
- updatedAt is automatically set to current datetime when any task field is modified
- When a task is created without a category, it should be assigned to the default "Uncategorized" category
- If the associated category is deleted, the task should be moved to the default "Uncategorized" category
- Completed tasks remain accessible until explicitly deleted by user
- Deleted tasks are permanently removed (no soft delete)

---

### Category
**Description:** Represents a user-defined grouping or list for organizing related tasks together

**Attributes:**
| Attribute | Type | Required | Constraints | Description |
|-----------|------|----------|-------------|-------------|
| id | identifier | Yes | Primary Key, Unique | Unique identifier for the category |
| name | string | Yes | Min 1 char, Unique | The display name of the category |
| isDefault | boolean | Yes | Default: false | Indicates if this is the system default "Uncategorized" category |
| createdAt | datetime | Yes | Auto-generated on creation | Timestamp when the category was created |

**Business Rules:**
- Category names must be unique (no duplicate category names allowed)
- Category name cannot be empty or whitespace only
- A default "Uncategorized" category must exist in the system (isDefault: true)
- The default category cannot be deleted or renamed
- When a category is deleted, all tasks in that category are moved to the default category
- createdAt is automatically set to current datetime when category is created

# Relationships
#### Task ↔ Category
**Type:** Many-to-One

**Description:** Tasks can be organized into categories for grouping related items together. Each task belongs to exactly one category, while each category can contain multiple tasks. This relationship supports the task categorization feature (FR-9) allowing users to organize tasks into custom lists. When a task is created without an explicit category assignment, it is placed in the default "Uncategorized" category. If a category is deleted, all its associated tasks are automatically reassigned to the default category.

**Cardinality:**
- Task: 0..* (each category can contain zero or more tasks)
- Category: 1..1 (each task belongs to exactly one category - required, defaults to "Uncategorized")

**Implementation Notes:**
- Foreign key: Task.categoryId → Category.id
- The Task table owns the relationship through the categoryId column
- Relationship is required on the Task side (tasks must have a category)
- When categoryId is not specified during task creation, system assigns the default category
- When a Category is deleted, all Tasks with that categoryId must be updated to reference the default category's id
- The default category (isDefault: true) cannot be deleted, ensuring referential integrity

---