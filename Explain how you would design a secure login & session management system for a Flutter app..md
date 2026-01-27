We’ll start with the **backend (Django + Django REST Framework + JWT)** for authentication and session management, then move to the **Flutter side** for secure token storage and handling refresh logic.

---

## 🧩 **PART 1: Backend — Django REST Framework + JWT Authentication**

### **1. Install dependencies**

```bash
pip install djangorestframework djangorestframework-simplejwt django-cors-headers
```

---

### **2. Update `settings.py`**

Add apps and middleware:

```python
INSTALLED_APPS = [
    'rest_framework',
    'rest_framework_simplejwt.token_blacklist',
    'corsheaders',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    # ... your other apps
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    # ... other middleware
]

CORS_ALLOW_ALL_ORIGINS = True  # For dev only — restrict in production
```

Configure DRF to use SimpleJWT:

```python
from datetime import timedelta

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    )
}

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=15),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'AUTH_HEADER_TYPES': ('Bearer',),
}
```

---

### **3. Create `auth_app/views.py`**

```python
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework import status
from rest_framework_simplejwt.tokens import RefreshToken

class LoginView(TokenObtainPairView):
    """Handles user login and returns JWT tokens"""
    pass  # SimpleJWT already provides logic

class RefreshTokenView(TokenRefreshView):
    """Handles token refresh"""
    pass

class LogoutView(APIView):
    """Blacklist the refresh token upon logout"""
    permission_classes = [IsAuthenticated]

    def post(self, request):
        try:
            refresh_token = request.data["refresh"]
            token = RefreshToken(refresh_token)
            token.blacklist()
            return Response(status=status.HTTP_205_RESET_CONTENT)
        except Exception as e:
            return Response({"error": str(e)}, status=status.HTTP_400_BAD_REQUEST)
```

---

### **4. Create `auth_app/urls.py`**

```python
from django.urls import path
from .views import LoginView, RefreshTokenView, LogoutView

urlpatterns = [
    path('login/', LoginView.as_view(), name='token_obtain_pair'),
    path('refresh/', RefreshTokenView.as_view(), name='token_refresh'),
    path('logout/', LogoutView.as_view(), name='token_blacklist'),
]
```

Then include in your main `urls.py`:

```python
from django.urls import path, include

urlpatterns = [
    path('api/auth/', include('auth_app.urls')),
]
```

---

### **5. Test your endpoints**

**Login:**

```bash
POST /api/auth/login/
{
    "username": "testuser",
    "password": "password123"
}
```

✅ Response:

```json
{
    "access": "<ACCESS_TOKEN>",
    "refresh": "<REFRESH_TOKEN>"
}
```

**Refresh token:**

```bash
POST /api/auth/refresh/
{
    "refresh": "<REFRESH_TOKEN>"
}
```

**Logout:**

```bash
POST /api/auth/logout/
Authorization: Bearer <ACCESS_TOKEN>
{
    "refresh": "<REFRESH_TOKEN>"
}
```

That’s it — the backend is now ready.

---

## 📱 **PART 2: Flutter — Secure Login & Token Management**

### **1. Install dependencies**

In `pubspec.yaml`:

```yaml
dependencies:
  dio: ^5.3.0
  flutter_secure_storage: ^9.0.0
```

---

### **2. Setup Secure Storage**

Create a `token_storage.dart` file:

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class TokenStorage {
  static const _storage = FlutterSecureStorage();
  static const _accessTokenKey = 'access_token';
  static const _refreshTokenKey = 'refresh_token';

  static Future<void> saveTokens(String access, String refresh) async {
    await _storage.write(key: _accessTokenKey, value: access);
    await _storage.write(key: _refreshTokenKey, value: refresh);
  }

  static Future<String?> getAccessToken() => _storage.read(key: _accessTokenKey);
  static Future<String?> getRefreshToken() => _storage.read(key: _refreshTokenKey);

  static Future<void> clear() async => await _storage.deleteAll();
}
```

---

### **3. Setup Dio Client with Token Interceptor**

Create a `dio_client.dart`:

```dart
import 'package:dio/dio.dart';
import 'token_storage.dart';

class ApiClient {
  final Dio dio = Dio(BaseOptions(baseUrl: 'http://localhost:8000/api/'));

  ApiClient() {
    dio.interceptors.add(InterceptorsWrapper(
      onRequest: (options, handler) async {
        final token = await TokenStorage.getAccessToken();
        if (token != null) {
          options.headers['Authorization'] = 'Bearer $token';
        }
        return handler.next(options);
      },
      onError: (error, handler) async {
        if (error.response?.statusCode == 401) {
          // Try to refresh token
          final refreshToken = await TokenStorage.getRefreshToken();
          if (refreshToken != null) {
            try {
              final response = await dio.post(
                'auth/refresh/',
                data: {'refresh': refreshToken},
              );
              final newAccess = response.data['access'];
              await TokenStorage.saveTokens(newAccess, refreshToken);

              // Retry the original request
              error.requestOptions.headers['Authorization'] = 'Bearer $newAccess';
              final cloneReq = await dio.fetch(error.requestOptions);
              return handler.resolve(cloneReq);
            } catch (_) {
              await TokenStorage.clear();
            }
          }
        }
        return handler.next(error);
      },
    ));
  }
}
```

---

### **4. Authentication Functions**

```dart
import 'package:dio/dio.dart';
import 'token_storage.dart';
import 'dio_client.dart';

class AuthService {
  final dio = ApiClient().dio;

  Future<bool> login(String username, String password) async {
    try {
      final response = await dio.post(
        'auth/login/',
        data: {'username': username, 'password': password},
      );
      await TokenStorage.saveTokens(response.data['access'], response.data['refresh']);
      return true;
    } catch (e) {
      print('Login error: $e');
      return false;
    }
  }

  Future<void> logout() async {
    final refreshToken = await TokenStorage.getRefreshToken();
    if (refreshToken != null) {
      await dio.post('auth/logout/', data: {'refresh': refreshToken});
    }
    await TokenStorage.clear();
  }
}
```

---

### **5. Usage Example (Login Screen)**

```dart
final authService = AuthService();

void handleLogin() async {
  bool success = await authService.login('john', 'password123');
  if (success) {
    print('Login successful!');
  } else {
    print('Invalid credentials.');
  }
}
```

---

## ✅ **End-to-End Flow Summary**

1. **Login** → Backend validates → Returns `access` + `refresh` tokens.
    
2. **Flutter** securely stores tokens using `flutter_secure_storage`.
    
3. **API Calls** include `Authorization: Bearer <access>` automatically.
    
4. When **401 Unauthorized**, Flutter automatically refreshes the token.
    
5. **Logout** clears storage and blacklists refresh token on backend.
    

---

Would you like me to add **device-based refresh token binding** (to prevent token reuse across devices)?