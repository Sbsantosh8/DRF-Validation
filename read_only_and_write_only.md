# Django Rest Framework: `read_only` vs `write_only` & Django ORM `get_or_create`

A concise and easy-to-understand note for DRF serializer fields and Django ORM `get_or_create` method.

---

## 1. `read_only`

* **Purpose:** Field is **only included in responses**, not accepted in input.
* **Usage:** `read_only=True`
* **Effect:**

  * **GET response:** Field is included.
  * **POST/PUT payload:** Field **cannot** be passed.

**Example:**

```python
class EmployeeSerializer(serializers.ModelSerializer):
    department = DepartmentSerializer(read_only=True)
```

* **GET response:**

```json
{
  "id": 1,
  "first_name": "Alice",
  "last_name": "Smith",
  "department": {
      "id": 2,
      "name": "IT"
  }
}
```

* **POST payload:** Cannot include nested object for `department`. Only FK ID if you want to create.

---

## 2. `write_only`

* **Purpose:** Field is **only accepted in input**, not returned in responses.
* **Usage:** `write_only=True`
* **Effect:**

  * **POST/PUT payload:** Field can be included.
  * **GET response:** Field is **not returned**.

**Example:**

```python
class UserSerializer(serializers.ModelSerializer):
    password = serializers.CharField(write_only=True)
```

* **POST payload:**

```json
{
  "username": "john",
  "password": "secret123"
}
```

* **GET response:**

```json
{
  "id": 1,
  "username": "john"
  // password NOT included
}
```

---

## Summary Table

| Property          | Included in Response | Accepted in Input |
| ----------------- | -------------------- | ----------------- |
| `read_only=True`  | ✅ Yes                | ❌ No              |
| `write_only=True` | ❌ No                 | ✅ Yes             |

---

## Tips

* `read_only` is often used for **nested serializers**, IDs that shouldn’t change, or auto-generated fields.
* `write_only` is often used for **sensitive data** like passwords or tokens.
* Using these correctly helps **control the flow of data** in your APIs and avoid accidental updates of sensitive fields.

---

## 3. Django ORM `get_or_create`

* **Purpose:** Either **fetch an existing object** or **create a new one** if it doesn’t exist.
* **Usage:**

```python
obj, created = Model.objects.get_or_create(
    defaults=None,
    **lookup_fields
)
```

* **Parameters:**

  * `**lookup_fields` → Fields used to check if object exists.
  * `defaults` → Dict of values used to create the object if it doesn’t exist.
* **Returns:** Tuple `(obj, created)`

  * `obj` → Retrieved or newly created object.
  * `created` → `True` if created, `False` if fetched.

**Example 1: Simple usage**

```python
employee, created = Employee.objects.get_or_create(
    email="john@example.com",
    defaults={
        "first_name": "John",
        "last_name": "Doe",
        "job_title": "Developer",
    }
)

if created:
    print("New employee created")
else:
    print("Employee already exists")
```

**Example 2: Without `defaults`**

```python
category, created = Category.objects.get_or_create(name="Technology")
```

* Creates only with the `name` field if required fields allow it.

**Notes:**

* Always provide `defaults` for required fields not in the lookup.
* Wrapped in a **transaction** → safe from race conditions.
* Use `update_or_create()` if you want to **update fields if object exists**.
