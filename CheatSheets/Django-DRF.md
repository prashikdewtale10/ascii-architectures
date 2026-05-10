# Django REST Framework (DRF) Complete Cheat Sheet

### 1. DRF CORE ARCHITECTURE
```bash
Client
  │
  ▼
URL Router
  │
  ▼
APIView / ViewSet
  │
  ├── Authentication
  ├── Permissions
  ├── Throttling
  ├── Parsing
  │
  ▼
Serializer
  │
  ├── Validation
  ├── Transformation
  │
  ▼
Business Logic
  │
  ▼
Django ORM
  │
  ▼
Database
  │
  ▼
Response Serializer
  │
  ▼
Renderer
  │
  ▼
JSON Response
```
### 2. DRF PROJECT STRUCTURE
```bash
project/
│
├── manage.py
├── requirements.txt
│
├── config/
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
│
├── apps/
│   └── users/
│       ├── models.py
│       ├── serializers.py
│       ├── views.py
│       ├── urls.py
│       ├── services.py
│       ├── permissions.py
│       ├── filters.py
│       ├── tasks.py
│       ├── tests/
│       └── migrations/
│
├── common/
│   ├── exceptions.py
│   ├── pagination.py
│   ├── authentication.py
│   └── utils.py
│
└── docker/
```

### 3. INSTALLATION
```bash
pip install djangorestframework
```
```python
# settings.py
INSTALLED_APPS = [
    "rest_framework",
]
```

### 4. BASIC APIVIEW
```python
from rest_framework.views import APIView
from rest_framework.response import Response

class HelloView(APIView):

    def get(self, request):
        return Response({"msg": "hello"})
```
```python
# urls.py
from django.urls import path
from .views import HelloView

urlpatterns = [
    path("hello/", HelloView.as_view())
]
```
### 5. FUNCTION BASED API
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(["GET"])
def hello(request):
    return Response({"msg": "hello"})
```

### 6. REQUEST OBJECT
```python
request.data
request.query_params
request.user
request.auth
request.headers
request.method
request.FILES
```

### 7. RESPONSE OBJECT
```python
from rest_framework.response import Response

return Response(
    data={"msg": "success"},
    status=200
)
```

### 8. STATUS CODES
```python
from rest_framework import status

status.HTTP_200_OK
status.HTTP_201_CREATED
status.HTTP_400_BAD_REQUEST
status.HTTP_401_UNAUTHORIZED
status.HTTP_403_FORBIDDEN
status.HTTP_404_NOT_FOUND
status.HTTP_500_INTERNAL_SERVER_ERROR
```
### 9. SERIALIZERS
#### Basic Serializer
```python
from rest_framework import serializers

class UserSerializer(serializers.Serializer):
    name = serializers.CharField()
    age = serializers.IntegerField()

```
#### Model Serializer
```python
from rest_framework import serializers
from .models import User

class UserSerializer(serializers.ModelSerializer):

    class Meta:
        model = User
        fields = "__all__"
```

### 10. SERIALIZER VALIDATION FLOW
```bash
Incoming JSON
      │
      ▼
to_internal_value()
      │
      ▼
Field Validators
      │
      ▼
validate_<field>()
      │
      ▼
validate()
      │
      ▼
validated_data
```

### 11. FIELD VALIDATION
```python
def validate_age(self, value):

    if value < 18:
        raise serializers.ValidationError(
            "Age must be >= 18"
        )

    return value
```

### 12. OBJECT LEVEL VALIDATION
```python
def validate(self, attrs):

    if attrs["start"] > attrs["end"]:
        raise serializers.ValidationError(
            "Invalid range"
        )

    return attrs
```
