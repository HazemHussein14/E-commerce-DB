# Categories Table Denormalization

### Overview

The traditional design of the Categories table involves a one-to-many recursive relationship, where each category references its parent category. While this structure is normalized and efficient for data integrity, it poses challenges for retrieving hierarchical data, requiring recursive queries or multiple joins that can be complex and slow.

To address this, a denormalized approach using a JSON field to store child categories has been adopted. This solution simplifies data retrieval by encapsulating hierarchical relationships directly within the parent record, eliminating the need for recursion and improving query performance.

## Table Schema

### DDL: Create Table

```sql
CREATE TABLE Categories (
    CategoryID SERIAL PRIMARY KEY,
    CategoryName CHAR(200) NOT NULL,
    ParentCategory INT REFERENCES Categories(CategoryID),
    DepartmentID INT NOT NULL,
    Children JSONB DEFAULT '[]'
);
```

### Description of Columns

- **CategoryID**: Unique identifier for each category.
- **CategoryName**: Name of the category.
- **ParentCategory**: References the `CategoryID` of the parent category.
- **DepartmentID**: Identifies the department the category belongs to.
- **Children**: JSONB field that stores child categories as an array of JSON objects.

---

## Sample Data

### Insert Parent Categories

```sql
INSERT INTO Categories (CategoryName, ParentCategory, DepartmentID)
VALUES
('Running Shoes', NULL, 1),
('Hiking Boots', NULL, 1),
('Sweatshirts', NULL, 2),
('Tracksuits', NULL, 2);
```

### Insert Child Categories

```sql
INSERT INTO Categories (CategoryName, ParentCategory, DepartmentID)
VALUES
('Hooded Sweats', 3, 2),
('V Neck Sweats', 3, 2),
('College Teams', 5, 2);
```

---

## Populating the `Children` Field

### SQL Query to Populate the JSON Field

```sql
UPDATE Categories AS parent
SET Children = (
    SELECT jsonb_agg(
        jsonb_build_object(
            'id', child.CategoryID,
            'name', child.CategoryName
        )
    )
    FROM Categories AS child
    WHERE child.ParentCategory = parent.CategoryID
)
WHERE EXISTS (
    SELECT 1
    FROM Categories AS child
    WHERE child.ParentCategory = parent.CategoryID
);
```

### Explanation

- **`jsonb_build_object`**: Creates a JSON object with `id` and `name` fields for each child category.
- **`jsonb_agg`**: Aggregates the JSON objects into a JSON array.
- The `Children` field is updated only for categories that have child entries.

---

## Querying the Table

### 1. Retrieve a Specific Category and Its Children

```sql
SELECT
    CategoryName,
    Children
FROM Categories
WHERE CategoryName = 'Sweatshirts';
```

**Sample Result:**

```
CategoryName  | Children
--------------|-----------------------------------------------------
Sweatshirts   | [{"id": 5, "name": "Hooded Sweats"}, {"id": 6, "name": "V Neck Sweats"}]
```

---

### 2. Search for a Child Category in JSON

```sql
SELECT
    CategoryID,
    CategoryName
FROM Categories
WHERE jsonb_path_exists(
    Children,
    '$[*] ? (@.name == "Hooded Sweats")'
);
```

**Sample Result:**

```
CategoryID | CategoryName
-----------|----------------
3          | Sweatshirts
```

---

## Advantages of This Approach

1. **Encapsulation**:

   - The child relationships are stored directly within the parent record, simplifying data retrieval.

2. **Flexibility**:

   - JSON allows you to store hierarchical data of varying depths without altering the schema.

3. **Performance**:

   - Retrieving precomputed JSON is faster than recursive queries or joins.

4. **Ease of Maintenance**:

   - Modifications to the hierarchy affect only the JSON field, reducing complexity.

---

## Trade-Offs

1. **Storage Overhead**:

   - JSON fields can consume more storage compared to normalized structures.

2. **Complex Updates**:

   - Updating a child’s information or reassigning relationships requires custom logic to update the JSON fields.

3. **Indexing Limitations**:

   - JSON fields may require specialized indexing (e.g., GIN indexes in PostgreSQL) to optimize searches.
