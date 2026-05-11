# Django REST Framework (DRF) Complete Cheat Sheet

### DRF CORE ARCHITECTURE
```bash
Client
  │
  ▼
Request
  │
  ▼
Middleware
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
```bash
pip install djangorestframework-simplejwt
```
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

    def has_object_permission(self, request, view, obj):
        return obj.user == request.user
```

### THROTTLING
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day'
    }
}
```

### FILTERING
```python
queryset = User.objects.filter(active=True)
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

### DJANGO FILTER
```bash
pip install django-filter
```
```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_FILTER_BACKENDS": [
        "django_filters.rest_framework.DjangoFilterBackend"
    ]
}
```
```python
# views.py
filterset_fields = ["status"]
```

### PAGINATION
#### Page Number Pagination
```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS":
        "rest_framework.pagination.PageNumberPagination",

    "PAGE_SIZE": 10
}
```

#### Custom Pagination
```python
from rest_framework.pagination import PageNumberPagination

class CustomPagination(PageNumberPagination):

    page_size = 20
```
#### FILE UPLOAD
```python
class UploadView(APIView):
    parser_classes = [MultiPartParser]

    def post(self, request):
        file = request.FILES["file"]
        return Response({"ok": True})
```

#### PARSERS
```
JSONParser
FormParser
MultiPartParser
FileUploadParser
```

#### RENDERERS
```
JSONRenderer
BrowsableAPIRenderer
```
#### DATABASE TRANSACTIONS
```python
from django.db import transaction

@transaction.atomic
def create_order():

    pass
```

### ORM OPTIMIZATION
#### select_related
```python
# Join Query
Order.objects.select_related("user")
```

#### prefetch_related
```python
# Separate optimized queries.
Order.objects.prefetch_related("items")
```

#### N+1 QUERY PROBLEM
```python
# Bad practice:
for order in orders:
    print(order.user.name)

# Good practice:
orders = Order.objects.select_related("user")
```

#### QUERYSET LAZY EVALUATION
```python
qs = User.objects.filter(active=True)
# NO SQL YET.
# SQL executes when:
list(qs)
len(qs)

for x in qs:
    pass
```

### CACHING
#### Per View Cache
```python
from django.views.decorators.cache import cache_page

@cache_page(60)
def my_view():
    pass
```

#### Redis Cache
```python
CACHES = {
    "default": {
        "BACKEND":
            "django_redis.cache.RedisCache",

        "LOCATION":
            "redis://redis:6379/1",
    }
}
```

### ASYNC VIEWS
```python
class AsyncView(APIView):

    async def get(self, request):
        return Response({"ok": True})
```
### CELERY IN DRF
```python
send_email.delay(user_id)
```
```
Used for:

Emails
PDF generation
Notifications
Reports
Heavy processing
```

### SIGNALS
```python
from django.db.models.signals import post_save

@receiver(post_save, sender=User)
def user_created(sender, instance, created, **kwargs):
    pass
```

### TESTING API
```python
from rest_framework.test import APITestCase

class UserTest(APITestCase):

    def test_create_user(self):
        response = self.client.post(
            "/users/",
            {"name": "abc"}
        )

        self.assertEqual(
            response.status_code,
            201
        )
```
### API CLIENT
```python
self.client.get()
self.client.post()
self.client.put()
self.client.patch()
self.client.delete()
```
