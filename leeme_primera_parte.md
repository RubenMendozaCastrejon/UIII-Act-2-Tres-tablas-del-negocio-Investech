Te ayudo a crear el proyecto Investech paso a paso. Voy a organizar todo el proceso de manera estructurada:

## 1. Estructura inicial del proyecto

```bash
# Crear carpeta del proyecto
mkdir UIII_investech_0640
cd UIII_investech_0640

# Abrir VS Code (esto lo haces manualmente desde el explorador de archivos)
# O desde terminal: code .
```

## 2. Configuración del entorno virtual

```bash
# Crear entorno virtual
python -m venv .venv

# Activar entorno virtual
# Windows:
.venv\Scripts\activate
# Linux/Mac:
source .venv/bin/activate

# Instalar Django
pip install django
```

## 3. Creación del proyecto Django

```bash
# Crear proyecto (asegúrate de estar en la carpeta UIII_investech_0640)
django-admin startproject backend_investech .

# Crear aplicación
python manage.py startapp app_investech
```

## 4. Configuración de settings.py

**backend_investech/settings.py**:
```python
"""
Django settings for backend_investech project.
"""

from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = 'django-investech-0640-ruben-mendoza-castregon'

DEBUG = True

ALLOWED_HOSTS = []

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_investech',  # Agregar nuestra aplicación
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'backend_investech.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'backend_investech.wsgi.application'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

LANGUAGE_CODE = 'es-mx'
TIME_ZONE = 'America/Mexico_City'
USE_I18N = True
USE_TZ = True

STATIC_URL = 'static/'

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

## 5. Modelos (models.py)

**app_investech/models.py**:
```python
from django.db import models

class Usuario(models.Model):
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    telefono = models.CharField(max_length=20, blank=True, null=True)
    fecha_registro = models.DateTimeField(auto_now_add=True)
    pais = models.CharField(max_length=50)
    saldo_disponible = models.DecimalField(max_digits=12, decimal_places=2, default=0.00)

    def __str__(self):
        return f"{self.nombre} {self.apellido}"

class Portafolio(models.Model):
    usuario = models.ForeignKey(Usuario, on_delete=models.CASCADE, related_name="portafolios")
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField(blank=True)
    valor_total = models.DecimalField(max_digits=12, decimal_places=2, default=0.00)
    fecha_creacion = models.DateTimeField(auto_now_add=True)
    riesgo = models.CharField(max_length=50, choices=[('bajo', 'Bajo'), ('medio', 'Medio'), ('alto', 'Alto')])
    activos = models.ManyToManyField('Activo', related_name="portafolios")

    def __str__(self):
        return f"{self.nombre} ({self.usuario.nombre})"

class Activo(models.Model):
    nombre = models.CharField(max_length=100)
    tipo = models.CharField(max_length=50, choices=[
        ('accion', 'Acción'),
        ('bono', 'Bono'),
        ('cripto', 'Criptomoneda'),
        ('fondo', 'Fondo de inversión'),
        ('otro', 'Otro'),
    ])
    simbolo = models.CharField(max_length=10)
    precio_actual = models.DecimalField(max_digits=10, decimal_places=2)
    fecha_actualizacion = models.DateTimeField(auto_now=True)
    mercado = models.CharField(max_length=100)
    volatilidad = models.DecimalField(max_digits=5, decimal_places=2, help_text="Porcentaje de volatilidad")

    def __str__(self):
        return f"{self.nombre} ({self.simbolo})"
```

## 6. Vistas (views.py)

**app_investech/views.py**:
```python
from django.shortcuts import render, redirect, get_object_or_404
from django.http import HttpResponse
from .models import Usuario
from datetime import datetime

def inicio_usuario(request):
    usuarios = Usuario.objects.all()
    return render(request, 'usuario/ver_usuario.html', {'usuarios': usuarios})

def agregar_usuario(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre')
        apellido = request.POST.get('apellido')
        email = request.POST.get('email')
        telefono = request.POST.get('telefono')
        pais = request.POST.get('pais')
        saldo_disponible = request.POST.get('saldo_disponible', 0.00)
        
        usuario = Usuario(
            nombre=nombre,
            apellido=apellido,
            email=email,
            telefono=telefono,
            pais=pais,
            saldo_disponible=saldo_disponible
        )
        usuario.save()
        return redirect('inicio_usuario')
    
    return render(request, 'usuario/agregar_usuario.html')

def actualizar_usuario(request, usuario_id):
    usuario = get_object_or_404(Usuario, id=usuario_id)
    return render(request, 'usuario/actualizar_usuario.html', {'usuario': usuario})

def realizar_actualizacion_usuario(request, usuario_id):
    if request.method == 'POST':
        usuario = get_object_or_404(Usuario, id=usuario_id)
        usuario.nombre = request.POST.get('nombre')
        usuario.apellido = request.POST.get('apellido')
        usuario.email = request.POST.get('email')
        usuario.telefono = request.POST.get('telefono')
        usuario.pais = request.POST.get('pais')
        usuario.saldo_disponible = request.POST.get('saldo_disponible', 0.00)
        usuario.save()
        return redirect('inicio_usuario')
    
    return redirect('inicio_usuario')

def borrar_usuario(request, usuario_id):
    usuario = get_object_or_404(Usuario, id=usuario_id)
    if request.method == 'POST':
        usuario.delete()
        return redirect('inicio_usuario')
    
    return render(request, 'usuario/borrar_usuario.html', {'usuario': usuario})

def inicio(request):
    return render(request, 'inicio.html')
```

## 7. URLs de la aplicación

**app_investech/urls.py**:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio, name='inicio'),
    path('usuarios/', views.inicio_usuario, name='inicio_usuario'),
    path('usuarios/agregar/', views.agregar_usuario, name='agregar_usuario'),
    path('usuarios/actualizar/<int:usuario_id>/', views.actualizar_usuario, name='actualizar_usuario'),
    path('usuarios/realizar_actualizacion/<int:usuario_id>/', views.realizar_actualizacion_usuario, name='realizar_actualizacion_usuario'),
    path('usuarios/borrar/<int:usuario_id>/', views.borrar_usuario, name='borrar_usuario'),
]
```

## 8. URLs del proyecto

**backend_investech/urls.py**:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_investech.urls')),
]
```

## 9. Admin (admin.py)

**app_investech/admin.py**:
```python
from django.contrib import admin
from .models import Usuario, Portafolio, Activo

@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'apellido', 'email', 'pais', 'saldo_disponible', 'fecha_registro')
    list_filter = ('pais', 'fecha_registro')
    search_fields = ('nombre', 'apellido', 'email')

@admin.register(Portafolio)
class PortafolioAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'usuario', 'valor_total', 'riesgo', 'fecha_creacion')
    list_filter = ('riesgo', 'fecha_creacion')

@admin.register(Activo)
class ActivoAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'simbolo', 'tipo', 'precio_actual', 'mercado')
    list_filter = ('tipo', 'mercado')
```

## 10. Templates

### base.html
**app_investech/templates/base.html**:
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Investech - Sistema de Administración</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        body {
            background-color: #f8f9fa;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }
        .main-content {
            flex: 1;
            padding: 20px 0;
        }
        .navbar-brand {
            font-weight: bold;
            color: #2c3e50 !important;
        }
        .bg-primary {
            background-color: #3498db !important;
        }
        .btn-primary {
            background-color: #3498db;
            border-color: #3498db;
        }
        .table-hover tbody tr:hover {
            background-color: rgba(52, 152, 219, 0.1);
        }
        .footer {
            background-color: #2c3e50;
            color: white;
            margin-top: auto;
        }
    </style>
</head>
<body>
    {% include 'header.html' %}
    {% include 'navbar.html' %}
    
    <div class="main-content">
        <div class="container">
            {% block content %}
            {% endblock %}
        </div>
    </div>
    
    {% include 'footer.html' %}
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### header.html
**app_investech/templates/header.html**:
```html
<header class="bg-primary text-white text-center py-3">
    <div class="container">
        <h1><i class="fas fa-chart-line me-2"></i>Sistema de Administración Investech</h1>
    </div>
</header>
```

### navbar.html
**app_investech/templates/navbar.html**:
```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container">
        <a class="navbar-brand" href="{% url 'inicio' %}">
            <i class="fas fa-chart-bar me-2"></i>Investech
        </a>
        
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>
        
        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav me-auto">
                <li class="nav-item">
                    <a class="nav-link" href="{% url 'inicio' %}">
                        <i class="fas fa-home me-1"></i>Inicio
                    </a>
                </li>
                
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="usuarioDropdown" role="button" data-bs-toggle="dropdown">
                        <i class="fas fa-users me-1"></i>Usuario
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="{% url 'agregar_usuario' %}">Agregar Usuario</a></li>
                        <li><a class="dropdown-item" href="{% url 'inicio_usuario' %}">Ver Usuario</a></li>
                        <li><hr class="dropdown-divider"></li>
                        <li><a class="dropdown-item" href="#">Actualizar Usuario</a></li>
                        <li><a class="dropdown-item" href="#">Borrar Usuario</a></li>
                    </ul>
                </li>
                
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="portafolioDropdown" role="button" data-bs-toggle="dropdown">
                        <i class="fas fa-briefcase me-1"></i>Portafolio
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="#">Agregar Portafolio</a></li>
                        <li><a class="dropdown-item" href="#">Ver Portafolio</a></li>
                        <li><hr class="dropdown-divider"></li>
                        <li><a class="dropdown-item" href="#">Actualizar Portafolio</a></li>
                        <li><a class="dropdown-item" href="#">Borrar Portafolio</a></li>
                    </ul>
                </li>
                
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="activoDropdown" role="button" data-bs-toggle="dropdown">
                        <i class="fas fa-coins me-1"></i>Activo
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="#">Agregar Activo</a></li>
                        <li><a class="dropdown-item" href="#">Ver Activo</a></li>
                        <li><hr class="dropdown-divider"></li>
                        <li><a class="dropdown-item" href="#">Actualizar Activo</a></li>
                        <li><a class="dropdown-item" href="#">Borrar Activo</a></li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</nav>
```

### footer.html
**app_investech/templates/footer.html**:
```html
<footer class="footer py-3 mt-5">
    <div class="container text-center">
        <p class="mb-1">&copy; 2024 Investech - Todos los derechos reservados</p>
        <p class="mb-1">Fecha del sistema: {% now "d/m/Y H:i" %}</p>
        <p class="mb-0">Creado por Ruben Mendoza Castrejon 0640, Cbtis 128</p>
    </div>
</footer>
```

### inicio.html
**app_investech/templates/inicio.html**:
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-8 mx-auto text-center">
        <h2 class="mb-4">Bienvenido a Investech</h2>
        <p class="lead mb-4">
            Sistema de administración de inversiones moderno y eficiente para gestionar 
            usuarios, portafolios y activos financieros.
        </p>
        
        <div class="row mt-5">
            <div class="col-md-4 mb-4">
                <div class="card h-100">
                    <div class="card-body">
                        <i class="fas fa-users fa-3x text-primary mb-3"></i>
                        <h5 class="card-title">Gestión de Usuarios</h5>
                        <p class="card-text">Administra los usuarios del sistema con operaciones CRUD completas.</p>
                    </div>
                </div>
            </div>
            
            <div class="col-md-4 mb-4">
                <div class="card h-100">
                    <div class="card-body">
                        <i class="fas fa-briefcase fa-3x text-success mb-3"></i>
                        <h5 class="card-title">Portafolios</h5>
                        <p class="card-text">Gestiona los portafolios de inversión de cada usuario.</p>
                    </div>
                </div>
            </div>
            
            <div class="col-md-4 mb-4">
                <div class="card h-100">
                    <div class="card-body">
                        <i class="fas fa-coins fa-3x text-warning mb-3"></i>
                        <h5 class="card-title">Activos</h5>
                        <p class="card-text">Administra los diferentes activos financieros disponibles.</p>
                    </div>
                </div>
            </div>
        </div>
        
        <div class="mt-5">
            <img src="https://images.unsplash.com/photo-1611974789855-9c2a0a7236a3?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1000&q=80" 
                 alt="Inversiones" class="img-fluid rounded shadow" style="max-height: 400px;">
        </div>
    </div>
</div>
{% endblock %}
```

## 11. Templates de Usuario

### agregar_usuario.html
**app_investech/templates/usuario/agregar_usuario.html**:
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-8 mx-auto">
        <div class="card">
            <div class="card-header bg-primary text-white">
                <h4 class="mb-0"><i class="fas fa-user-plus me-2"></i>Agregar Nuevo Usuario</h4>
            </div>
            <div class="card-body">
                <form method="POST">
                    {% csrf_token %}
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label for="nombre" class="form-label">Nombre</label>
                            <input type="text" class="form-control" id="nombre" name="nombre" required>
                        </div>
                        <div class="col-md-6 mb-3">
                            <label for="apellido" class="form-label">Apellido</label>
                            <input type="text" class="form-control" id="apellido" name="apellido" required>
                        </div>
                    </div>
                    
                    <div class="mb-3">
                        <label for="email" class="form-label">Email</label>
                        <input type="email" class="form-control" id="email" name="email" required>
                    </div>
                    
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label for="telefono" class="form-label">Teléfono</label>
                            <input type="text" class="form-control" id="telefono" name="telefono">
                        </div>
                        <div class="col-md-6 mb-3">
                            <label for="pais" class="form-label">País</label>
                            <input type="text" class="form-control" id="pais" name="pais" required>
                        </div>
                    </div>
                    
                    <div class="mb-3">
                        <label for="saldo_disponible" class="form-label">Saldo Disponible</label>
                        <input type="number" class="form-control" id="saldo_disponible" name="saldo_disponible" step="0.01" value="0.00">
                    </div>
                    
                    <div class="d-grid gap-2 d-md-flex justify-content-md-end">
                        <a href="{% url 'inicio_usuario' %}" class="btn btn-secondary me-md-2">Cancelar</a>
                        <button type="submit" class="btn btn-primary">Guardar Usuario</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### ver_usuario.html
**app_investech/templates/usuario/ver_usuario.html**:
```html
{% extends 'base.html' %}

{% block content %}
<div class="d-flex justify-content-between align-items-center mb-4">
    <h2><i class="fas fa-users me-2"></i>Lista de Usuarios</h2>
    <a href="{% url 'agregar_usuario' %}" class="btn btn-primary">
        <i class="fas fa-plus me-1"></i>Agregar Usuario
    </a>
</div>

<div class="card">
    <div class="card-body">
        {% if usuarios %}
        <div class="table-responsive">
            <table class="table table-striped table-hover">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Apellido</th>
                        <th>Email</th>
                        <th>Teléfono</th>
                        <th>País</th>
                        <th>Saldo</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {% for usuario in usuarios %}
                    <tr>
                        <td>{{ usuario.id }}</td>
                        <td>{{ usuario.nombre }}</td>
                        <td>{{ usuario.apellido }}</td>
                        <td>{{ usuario.email }}</td>
                        <td>{{ usuario.telefono|default:"-" }}</td>
                        <td>{{ usuario.pais }}</td>
                        <td>${{ usuario.saldo_disponible }}</td>
                        <td>
                            <div class="btn-group btn-group-sm">
                                <a href="#" class="btn btn-info" title="Ver">
                                    <i class="fas fa-eye"></i>
                                </a>
                                <a href="{% url 'actualizar_usuario' usuario.id %}" class="btn btn-warning" title="Editar">
                                    <i class="fas fa-edit"></i>
                                </a>
                                <a href="{% url 'borrar_usuario' usuario.id %}" class="btn btn-danger" title="Borrar">
                                    <i class="fas fa-trash"></i>
                                </a>
                            </div>
                        </td>
                    </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
        {% else %}
        <div class="text-center py-4">
            <i class="fas fa-users fa-3x text-muted mb-3"></i>
            <h4>No hay usuarios registrados</h4>
            <p class="text-muted">Comienza agregando el primer usuario al sistema.</p>
            <a href="{% url 'agregar_usuario' %}" class="btn btn-primary">Agregar Primer Usuario</a>
        </div>
        {% endif %}
    </div>
</div>
{% endblock %}
```

### actualizar_usuario.html
**app_investech/templates/usuario/actualizar_usuario.html**:
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-8 mx-auto">
        <div class="card">
            <div class="card-header bg-warning text-dark">
                <h4 class="mb-0"><i class="fas fa-user-edit me-2"></i>Actualizar Usuario</h4>
            </div>
            <div class="card-body">
                <form method="POST" action="{% url 'realizar_actualizacion_usuario' usuario.id %}">
                    {% csrf_token %}
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label for="nombre" class="form-label">Nombre</label>
                            <input type="text" class="form-control" id="nombre" name="nombre" value="{{ usuario.nombre }}" required>
                        </div>
                        <div class="col-md-6 mb-3">
                            <label for="apellido" class="form-label">Apellido</label>
                            <input type="text" class="form-control" id="apellido" name="apellido" value="{{ usuario.apellido }}" required>
                        </div>
                    </div>
                    
                    <div class="mb-3">
                        <label for="email" class="form-label">Email</label>
                        <input type="email" class="form-control" id="email" name="email" value="{{ usuario.email }}" required>
                    </div>
                    
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label for="telefono" class="form-label">Teléfono</label>
                            <input type="text" class="form-control" id="telefono" name="telefono" value="{{ usuario.telefono|default:'' }}">
                        </div>
                        <div class="col-md-6 mb-3">
                            <label for="pais" class="form-label">País</label>
                            <input type="text" class="form-control" id="pais" name="pais" value="{{ usuario.pais }}" required>
                        </div>
                    </div>
                    
                    <div class="mb-3">
                        <label for="saldo_disponible" class="form-label">Saldo Disponible</label>
                        <input type="number" class="form-control" id="saldo_disponible" name="saldo_disponible" step="0.01" value="{{ usuario.saldo_disponible }}">
                    </div>
                    
                    <div class="d-grid gap-2 d-md-flex justify-content-md-end">
                        <a href="{% url 'inicio_usuario' %}" class="btn btn-secondary me-md-2">Cancelar</a>
                        <button type="submit" class="btn btn-warning">Actualizar Usuario</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### borrar_usuario.html
**app_investech/templates/usuario/borrar_usuario.html**:
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-6 mx-auto">
        <div class="card">
            <div class="card-header bg-danger text-white">
                <h4 class="mb-0"><i class="fas fa-user-times me-2"></i>Eliminar Usuario</h4>
            </div>
            <div class="card-body text-center">
                <i class="fas fa-exclamation-triangle fa-3x text-warning mb-3"></i>
                <h5>¿Estás seguro de que deseas eliminar este usuario?</h5>
                <p class="text-muted">Esta acción no se puede deshacer.</p>
                
                <div class="card mb-3">
                    <div class="card-body">
                        <h6>{{ usuario.nombre }} {{ usuario.apellido }}</h6>
                        <p class="mb-1"><strong>Email:</strong> {{ usuario.email }}</p>
                        <p class="mb-1"><strong>País:</strong> {{ usuario.pais }}</p>
                        <p class="mb-0"><strong>Saldo:</strong> ${{ usuario.saldo_disponible }}</p>
                    </div>
                </div>
                
                <form method="POST">
                    {% csrf_token %}
                    <div class="d-grid gap-2 d-md-flex justify-content-md-center">
                        <a href="{% url 'inicio_usuario' %}" class="btn btn-secondary me-md-2">Cancelar</a>
                        <button type="submit" class="btn btn-danger">Sí, Eliminar Usuario</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

## 12. Comandos finales para ejecutar

```bash
# Realizar migraciones
python manage.py makemigrations
python manage.py migrate

# Crear superusuario (opcional)
python manage.py createsuperuser

# Ejecutar servidor en puerto 8640
python manage.py runserver 8640
```

## 13. Estructura final de carpetas

```
UIII_investech_0640/
├── .venv/
├── backend_investech/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── app_investech/
│   ├── migrations/
│   ├── templates/
│   │   ├── base.html
│   │   ├── header.html
│   │   ├── navbar.html
│   │   ├── footer.html
│   │   ├── inicio.html
│   │   └── usuario/
│   │       ├── agregar_usuario.html
│   │       ├── ver_usuario.html
│   │       ├── actualizar_usuario.html
│   │       └── borrar_usuario.html
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   ├── urls.py
│   └── views.py
├── db.sqlite3
└── manage.py
```

El proyecto estará completamente funcional en `http://localhost:8640` con todas las operaciones CRUD para el modelo Usuario, usando colores suaves y diseño moderno con Bootstrap.
