# Bookstore API

A RESTful API built with Django REST Framework that allows users to manage a bookstore inventory through CRUD operations.

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [API Endpoints](#api-endpoints)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Models](#models)
- [Serializers](#serializers)
- [ViewSets](#viewsets)
- [Authentication](#authentication)
- [Testing](#testing)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

## Overview

This Bookstore API provides a simple yet robust backend for managing a collection of books. It enables creating, reading, updating, and deleting book records through a RESTful interface. The project is built using Django REST Framework, making it scalable, maintainable, and easy to extend.

## Features

- Full CRUD functionality for book resources
- RESTful API design
- Automatic browsable API interface
- JSON response format
- Filtering, sorting, and pagination capabilities
- Authentication and permission controls
- Comprehensive endpoint documentation

## Tech Stack

- Python 3.8+
- Django 4.0+
- Django REST Framework 3.13+
- SQLite (default) / PostgreSQL (recommended for production)
- Django ORM

## API Endpoints

| Endpoint | HTTP Method | CRUD Operation | Description |
|----------|-------------|----------------|-------------|
| `/api/books/` | GET | READ | List all books |
| `/api/books/` | POST | CREATE | Create a new book |
| `/api/books/{id}/` | GET | READ | Retrieve a specific book |
| `/api/books/{id}/` | PUT | UPDATE | Update a specific book (full update) |
| `/api/books/{id}/` | PATCH | UPDATE | Update a specific book (partial update) |
| `/api/books/{id}/` | DELETE | DELETE | Delete a specific book |

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/bookstore-api.git
   cd bookstore-api
   ```

2. Create and activate a virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows, use: venv\Scripts\activate
   ```

3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

4. Run migrations:
   ```bash
   python manage.py migrate
   ```

5. Create a superuser (for admin access):
   ```bash
   python manage.py createsuperuser
   ```

6. Start the development server:
   ```bash
   python manage.py runserver
   ```

7. Access the API at `http://127.0.0.1:8000/api/books/`

## Configuration

### Environment Variables

Create a `.env` file in the project root with the following variables:

```
DEBUG=True
SECRET_KEY=your_secret_key
DATABASE_URL=sqlite:///db.sqlite3
ALLOWED_HOSTS=localhost,127.0.0.1
```

For production, update these values accordingly.

### Database Configuration

By default, the project uses SQLite. To configure a different database like PostgreSQL, update your settings:

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'bookstore',
        'USER': 'postgres',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

## Usage

### Using the API with cURL

**List all books:**
```bash
curl -X GET http://127.0.0.1:8000/api/books/
```

**Get a specific book:**
```bash
curl -X GET http://127.0.0.1:8000/api/books/1/
```

**Create a new book:**
```bash
curl -X POST http://127.0.0.1:8000/api/books/ \
  -H "Content-Type: application/json" \
  -d '{"title":"Django for Beginners", "author":"William S. Vincent", "isbn":"9781735467207", "price":39.99, "quantity": 15}'
```

**Update a book:**
```bash
curl -X PUT http://127.0.0.1:8000/api/books/1/ \
  -H "Content-Type: application/json" \
  -d '{"title":"Django for Professionals", "author":"William S. Vincent", "isbn":"9781735467214", "price":42.99, "quantity": 10}'
```

**Partially update a book:**
```bash
curl -X PATCH http://127.0.0.1:8000/api/books/1/ \
  -H "Content-Type: application/json" \
  -d '{"price":45.99, "quantity": 20}'
```

**Delete a book:**
```bash
curl -X DELETE http://127.0.0.1:8000/api/books/1/
```

### Using the Browsable API

You can also interact with the API through the browsable interface provided by Django REST Framework by navigating to `http://127.0.0.1:8000/api/books/` in your web browser.

## Models

The main model in this project is the `Book` model, which would typically look like:

```python
# models.py
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=100)
    isbn = models.CharField(max_length=13, unique=True)
    price = models.DecimalField(max_digits=6, decimal_places=2)
    quantity = models.IntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
```

## Serializers

Serializers convert Django model instances to Python datatypes that can be easily rendered into JSON:

```python
# serializers.py
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'isbn', 'price', 'quantity', 'created_at', 'updated_at']
```

## ViewSets

ViewSets handle the API logic and provide CRUD operations:

```python
# views.py
from rest_framework import viewsets
from .models import Book
from .serializers import BookSerializer

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

## Authentication

By default, the API uses Django REST Framework's session authentication. To enable token authentication:

1. Add `rest_framework.authtoken` to `INSTALLED_APPS` in `settings.py`
2. Run migrations: `python manage.py migrate`
3. Update settings:

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}
```

4. Generate a token for a user:

```bash
python manage.py drf_create_token username
```

5. Use the token in API requests:

```bash
curl -X GET http://127.0.0.1:8000/api/books/ -H "Authorization: Token your_token_here"
```

## Testing

Run tests with:

```bash
python manage.py test
```

Example test case:

```python
# tests.py
from django.test import TestCase
from django.urls import reverse
from rest_framework import status
from rest_framework.test import APIClient
from .models import Book

class BookAPITests(TestCase):
    def setUp(self):
        self.client = APIClient()
        self.book_data = {
            'title': 'Test Book',
            'author': 'Test Author',
            'isbn': '1234567890123',
            'price': 19.99,
            'quantity': 5
        }
        self.book = Book.objects.create(**self.book_data)

    def test_get_all_books(self):
        response = self.client.get(reverse('book-list'))
        self.assertEqual(response.status_code, status.HTTP_200_OK)

    def test_create_book(self):
        new_book = {
            'title': 'New Book',
            'author': 'New Author',
            'isbn': '3210987654321',
            'price': 29.99,
            'quantity': 10
        }
        response = self.client.post(reverse('book-list'), new_book, format='json')
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
```

## Deployment

### Preparing for Production

1. Update settings for production:
   ```python
   # settings.py
   DEBUG = False
   ALLOWED_HOSTS = ['your-domain.com']
   ```

2. Configure a production database
3. Set up static files serving
4. Use a WSGI server like Gunicorn
5. Set up a reverse proxy with Nginx

### Example Deployment with Docker

1. Create a `Dockerfile`:
   ```dockerfile
   FROM python:3.9-slim

   WORKDIR /app

   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt

   COPY . .

   EXPOSE 8000

   CMD ["gunicorn", "--bind", "0.0.0.0:8000", "bookstore.wsgi:application"]
   ```

2. Create a `docker-compose.yml`:
   ```yaml
   version: '3'

   services:
     web:
       build: .
       command: gunicorn bookstore.wsgi:application --bind 0.0.0.0:8000
       volumes:
         - .:/app
       ports:
         - "8000:8000"
       depends_on:
         - db
     db:
       image: postgres:13
       volumes:
         - postgres_data:/var/lib/postgresql/data/
       environment:
         - POSTGRES_PASSWORD=password
         - POSTGRES_USER=postgres
         - POSTGRES_DB=bookstore

   volumes:
     postgres_data:
   ```

3. Run with Docker Compose:
   ```bash
   docker-compose up -d
   ```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add some feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

Created with steve ongera