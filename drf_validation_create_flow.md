# Django REST Framework – Validation & Create/Update Notes

## 1️⃣ Model-Level Validation & Creation
- **Purpose:** Always enforce rules, independent of API.
- Methods:
  - `save()`
  - `clean()`
  - Custom Manager

```python
class Employee(models.Model):
    email = models.EmailField(unique=True)
    age = models.IntegerField()

    def save(self, *args, **kwargs):
        if self.age < 18:
            raise ValidationError("Age must be at least 18")
        super().save(*args, **kwargs)
```

```python
# Custom Manager
class EmployeeManager(models.Manager):
    def create_employee(self, email, age):
        if age < 18:
            raise ValueError("Must be adult")
        employee = self.model(email=email, age=age)
        employee.save()
        return employee

class Employee(models.Model):
    email = models.EmailField(unique=True)
    age = models.IntegerField()
    objects = EmployeeManager()
```

## 2️⃣ Serializer-Level Validation & Create/Update
- **Purpose:** Validate input from API, customize creation/update behavior.

### Field-level Validation
```python
    def validate_age(self, value):
        if value < 18:
            raise serializers.ValidationError("Employee must be adult")
        return value
```

### Object-level Validation
```python
    def validate(self, data):
        if data['age'] < 18 and data['email'].endswith('@company.com'):
            raise serializers.ValidationError("Underage cannot have company email")
        return data
```

### Create / Update Methods
```python
    def create(self, validated_data):
        employee = Employee.objects.create(**validated_data)
        return employee

    def update(self, instance, validated_data):
        instance.age = validated_data.get('age', instance.age)
        instance.save()
        return instance
```

## 3️⃣ View-Level Customizations
- **Purpose:** Context-specific logic, user/request-dependent actions.

### Generic View Example
```python
class EmployeeCreateAPIView(generics.CreateAPIView):
    serializer_class = EmployeeSerializer

    def perform_create(self, serializer):
        serializer.save(created_by=self.request.user)
```

### APIView Example
```python
class EmployeeAPIView(APIView):
    def post(self, request):
        serializer = EmployeeSerializer(data=request.data)
        if not serializer.is_valid():
            return Response({"errors": serializer.errors}, status=400)
        serializer.save()
        return Response(serializer.data, status=201)
```

## 4️⃣ Summary Table
| Layer | Method | Purpose |
|-------|--------|---------|
| Model | save() / clean() / Custom Manager | Universal validations, always enforced |
| Serializer | validate_<field>(), validate(), create(), update() | API input validation & logic |
| View | perform_create() / perform_update() | Contextual actions, request-specific logic |

## 5️⃣ Flow Diagram (SVG)
```svg
<svg width="700" height="350" xmlns="http://www.w3.org/2000/svg">
  <!-- API Request -->
  <rect x="20" y="20" width="180" height="60" fill="#e3f2fd" stroke="#1976d2" stroke-width="2" rx="10"/>
  <text x="40" y="55" font-family="Arial" font-size="14" font-weight="bold">API Request</text>

  <!-- Serializer Validation -->
  <rect x="250" y="20" width="200" height="60" fill="#e8f5e9" stroke="#2e7d32" stroke-width="2" rx="10"/>
  <text x="260" y="55" font-family="Arial" font-size="14" font-weight="bold">Serializer Validation</text>

  <!-- Model Validation -->
  <rect x="500" y="20" width="180" height="60" fill="#fff3e0" stroke="#f57c00" stroke-width="2" rx="10"/>
  <text x="520" y="55" font-family="Arial" font-size="14" font-weight="bold">Model save()/clean()</text>

  <!-- DB Save -->
  <rect x="250" y="120" width="200" height="60" fill="#f3e5f5" stroke="#6a1b9a" stroke-width="2" rx="10"/>
  <text x="300" y="155" font-family="Arial" font-size="14" font-weight="bold">Database</text>

  <!-- Arrows -->
  <line x1="200" y1="50" x2="250" y2="50" stroke="#000" stroke-width="2" marker-end="url(#arrow)" />
  <line x1="450" y1="50" x2="500" y2="50" stroke="#000" stroke-width="2" marker-end="url(#arrow)" />
  <line x1="600" y1="80" x2="350" y2="120" stroke="#000" stroke-width="2" marker-end="url(#arrow)" />

  <defs>
    <marker id="arrow" markerWidth="10" markerHeight="10" refX="0" refY="3" orient="auto">
      <path d="M0,0 L0,6 L9,3 z" fill="#000" />
    </marker>
  </defs>
</svg>
```
> Flow: **API Request → Serializer Validation → Model save()/clean() → Database**

## 6️⃣ Recommended Study Flow
1. Model validations → `clean()`, `save()`, `unique`, `validators`
2. Serializer validations → field-level, object-level, create/update
3. Views → `perform_create()` / `perform_update()` for context
4. Mini project → Employee CRUD with all layers
5. Review flow: **Model → Serializer → View → DB**

# DRF Create/Update Flow with Parameters

```text
[API Request] (JSON body)
     ↓
[View] 
  - passes request.data to serializer
     ↓
[Serializer.is_valid()]
  - Input: request.data
  - Output: validated_data (if valid)
     ↓
 ┌───────────────────────┐
 │ validate_<field>(value) │
 │ validate(attrs)         │
 └───────────────────────┘
     ↓
[Serializer.save()]
  - Input: validated_data
  - Decides → create() or update()
     ↓
 ┌───────────────────┐
 │ create(validated_data) │
 │ update(instance, validated_data) │
 └───────────────────┘
     ↓
[Model.save()]
  - Input: model instance (with fields set)
  - Commits data → DB
     ↓
[Database ✅]
