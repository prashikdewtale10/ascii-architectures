# Django REST Framework (DRF) Complete Cheat Sheet

### DRF CORE ARCHITECTURE
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
### DRF PROJECT STRUCTURE
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

### INSTALLATION
```bash
pip install djangorestframework
```
```python
# settings.py
INSTALLED_APPS = [
    "rest_framework",
]
```

### BASIC APIVIEW
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
### FUNCTION BASED API
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(["GET"])
def hello(request):
    return Response({"msg": "hello"})
```

### REQUEST OBJECT
```python
request.data
request.query_params
request.user
request.auth
request.headers
request.method
request.FILES
```

### RESPONSE OBJECT
```python
from rest_framework.response import Response

return Response(
    data={"msg": "success"},
    status=200
)
```

### STATUS CODES
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
### SERIALIZERS
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

### SERIALIZER VALIDATION FLOW
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

### FIELD VALIDATION
```python
def validate_age(self, value):

    if value < 18:
        raise serializers.ValidationError(
            "Age must be >= 18"
        )

    return value
```

### OBJECT LEVEL VALIDATION
```python
def validate(self, attrs):

    if attrs["start"] > attrs["end"]:
        raise serializers.ValidationError(
            "Invalid range"
        )

    return attrs
```

### CREATE / UPDATE METHODS
```python
def create(self, validated_data):
    return User.objects.create(**validated_data)

def update(self, instance, validated_data):

    instance.name = validated_data.get(
        "name",
        instance.name
    )

    instance.save()

    return instance
```

### GENERIC VIEWS
```python
from rest_framework.generics import ListAPIView

class UserList(ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```
### SERIALIZER IMPORTANT ATTRIBUTES
```python
serializer.data
serializer.errors
serializer.validated_data
serializer.initial_data
```

### COMMON GENERIC VIEWS

| View              | Purpose    |
| ----------------- | ---------- |
| ListAPIView       | GET list   |
| RetrieveAPIView   | GET single |
| CreateAPIView     | POST       |
| UpdateAPIView     | PUT/PATCH  |
| DestroyAPIView    | DELETE     |
| ListCreateAPIView | GET + POST |

### GENERIC API VIEWS
```python
from rest_framework.generics import (
    ListAPIView
)

class UserListView(ListAPIView):

    queryset = User.objects.all()
    serializer_class = UserSerializer
```

### VIEWSETS
```python
from rest_framework.viewsets import ModelViewSet

class UserViewSet(ModelViewSet):

    queryset = User.objects.all()
    serializer_class = UserSerializer
```

### ROUTERS
```python
from rest_framework.routers import DefaultRouter

router = DefaultRouter()

router.register(
    "users",
    UserViewSet
)

urlpatterns = router.urls
```

### VIEWSET METHOD MAPPING
```python
GET       /users/        → list
GET       /users/1/      → retrieve
POST      /users/        → create
PUT       /users/1/      → update
PATCH     /users/1/      → partial_update
DELETE    /users/1/      → destroy
```

### CUSTOM ACTIONS
```python
from rest_framework.decorators import action

@action(detail=True, methods=["post"])
def activate(self, request, pk=None):

    return Response({"status": "activated"})
```

### AUTHENTICATION
```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": (
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    )
}
```

### PERMISSIONS
#### Global Permission
```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated"
    ]
}
```
#### Per View

```python
from rest_framework.permissions import IsAdminUser

permission_classes = [IsAdminUser]
```

### CUSTOM PERMISSION
```python
from rest_framework.permissions import BasePermission

class IsOwner(BasePermission):

    def has_object_permission(
        self,
        request,
        view,
        obj
    ):
        return obj.user == request.user
```

### THROTTLING
```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_THROTTLE_CLASSES": [
        "rest_framework.throttling.UserRateThrottle"
    ],

    "DEFAULT_THROTTLE_RATES": {
        "user": "100/min"
    }
}
```

### FILTERING
```python
FILTERING
```
#### Search Filter
```python
from rest_framework.filters import SearchFilter

filter_backends = [SearchFilter]
search_fields = ["name"]
```

#### Ordering Filter
```python
from rest_framework.filters import OrderingFilter

filter_backends = [OrderingFilter]
ordering_fields = ["created_at"]
```
