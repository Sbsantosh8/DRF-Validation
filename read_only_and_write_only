# Django Rest Framework: `read_only` vs `write_only`

A concise and easy-to-understand note for DRF serializer fields.

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

This note can be directly used as reference while creating DRF serializers.
