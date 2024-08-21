

# Kanban (Trello clone)
## A task management application similar to Trello, featuring boards, lists, and cards with drag-and-drop functionality and real-time updates.

---

# Backend
# Step-by-Step Tutorial for Building "Kanban" using Django

This guide walks you through the process of developing a Trello clone using Django for the backend, focusing on models, views, and URL configurations. We'll also structure the frontend using HTML templates and integrate HTMX and AlpineJS for interactive features.

---

## 1. Project Setup

### Django Project and App Creation

1. **Create a Django Project:**

   ```bash
   django-admin startproject kanban
   cd kanban
   ```

2. **Create Django Apps:**

   Two apps are needed: `boards` for board management logic and `users` for user profiles and settings.

   ```bash
   python manage.py startapp boards
   python manage.py startapp users
   ```

3. **Settings Configuration:**

   - Add both apps to `INSTALLED_APPS` in `kanban/settings.py`:

     ```python
     INSTALLED_APPS = [
         ...
         'boards',
         'users',
     ]
     ```

4. **Database Setup:**

   Ensure a database is configured in `settings.py`. Defaults are sufficient for testing.

---

## 2. Backend Implementation

### Models

#### Boards App Models

1. **Model Definitions:**

   `boards/models.py`:

   ```python
   from django.db import models
   from django.contrib.auth.models import User

   class Board(models.Model):
       name = models.CharField(max_length=255)
       owner = models.ForeignKey(User, on_delete=models.CASCADE)
       created_at = models.DateTimeField(auto_now_add=True)

       def __str__(self):
           return self.name

   class List(models.Model):
       name = models.CharField(max_length=255)
       board = models.ForeignKey(Board, related_name='lists', on_delete=models.CASCADE)
       order = models.PositiveIntegerField()

       def __str__(self):
           return self.name

   class Card(models.Model):
       title = models.CharField(max_length=255)
       description = models.TextField(blank=True)
       list = models.ForeignKey(List, related_name='cards', on_delete=models.CASCADE)
       order = models.PositiveIntegerField()

       def __str__(self):
           return self.title
   ```

#### Users App Models

1. **User Profile and Settings:**

   `users/models.py`:

   ```python
   from django.db import models
   from django.contrib.auth.models import User

   class UserProfile(models.Model):
       user = models.OneToOneField(User, on_delete=models.CASCADE)
       bio = models.TextField(blank=True)

       def __str__(self):
           return self.user.username

   class UserSettings(models.Model):
       user = models.OneToOneField(User, on_delete=models.CASCADE)
       theme = models.CharField(max_length=20, default='light')

       def __str__(self):
           return f"Settings for {self.user.username}"
   ```

### Views

#### Boards App Views

1. **Views Definitions:**

   `boards/views.py`:

   ```python
   from django.views.generic import ListView, DetailView, UpdateView
   from .models import Board, List, Card

   class BoardListView(ListView):
       model = Board
       template_name = 'boards/board_list.html'
       context_object_name = 'boards'

   class BoardDetailView(DetailView):
       model = Board
       template_name = 'boards/board_detail.html'
       context_object_name = 'board'

   class CardUpdateView(UpdateView):
       model = Card
       fields = ['title', 'description', 'order']
       template_name = 'boards/card_form.html'
       success_url = '/dashboard/'
   ```

#### Users App Views

1. **Views Definitions:**

   `users/views.py`:

   ```python
   from django.views.generic import TemplateView, UpdateView
   from django.contrib.auth.decorators import login_required
   from .models import UserProfile, UserSettings

   class ProfileView(TemplateView):
       template_name = 'users/profile.html'
       
   class SettingsView(UpdateView):
       model = UserSettings
       fields = ['theme']
       template_name = 'users/settings_form.html'

       def get_object(self, queryset=None):
           return UserSettings.objects.get(user=self.request.user)
   ```

### URL Configurations

1. **Boards URL Patterns:**

   `boards/urls.py`:

   ```python
   from django.urls import path
   from .views import BoardListView, BoardDetailView, CardUpdateView

   urlpatterns = [
       path('', BoardListView.as_view(), name='board_list'),
       path('<int:pk>/', BoardDetailView.as_view(), name='board_detail'),
       path('card/<int:pk>/edit/', CardUpdateView.as_view(), name='card_update'),
   ]
   ```

2. **Users URL Patterns:**

   `users/urls.py`:

   ```python
   from django.urls import path
   from .views import ProfileView, SettingsView

   urlpatterns = [
       path('profile/', ProfileView.as_view(), name='profile'),
       path('settings/', SettingsView.as_view(), name='settings'),
   ]
   ```

3. **Main URL Configuration:**

   `kanban/urls.py`:

   ```python
   from django.contrib import admin
   from django.urls import path, include

   urlpatterns = [
       path('admin/', admin.site.urls),
       path('dashboard/', include('boards.urls')),
       path('users/', include('users.urls')),
   ]
   ```

---

## 3. Frontend Structure

### Base Template Outline

1. **`templates/base.html`:**

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Kanban - {% block title %}{% endblock %}</title>
       <!-- PLACEHOLDER: Insert CSS links here -->
   </head>
   <body>
       <!-- PLACEHOLDER: Navigation bar -->
       {% block content %}{% endblock %}
       <!-- PLACEHOLDER: Footer -->
       <!-- PLACEHOLDER: Insert JS scripts here -->
       {% block scripts %}{% endblock %}
   </body>
   </html>
   ```

### Page-Specific Templates

1. **Dashboard Page:**

   `templates/boards/board_list.html`:

   ```html
   {% extends "base.html" %}
   {% block title %}Dashboard{% endblock %}
   {% block content %}
   <div>
       <!-- PLACEHOLDER: CreateBoardButton -->
       <div>
           {% for board in boards %}
               <div>
                   <h2><a href="{% url 'board_detail' board.id %}">{{ board.name }}</a></h2>
               </div>
           {% endfor %}
       </div>
   </div>
   {% endblock %}
   ```

2. **Board Detail Page:**

   `templates/boards/board_detail.html`:

   ```html
   {% extends "base.html" %}
   {% block title %}{{ board.name }}{% endblock %}
   {% block content %}
   <div>
       <!-- PLACEHOLDER: AddListButton -->
       {% for list in board.lists.all %}
           <div>
               <h3>{{ list.name }}</h3>
               {% for card in list.cards.all %}
                   <div>
                       <h4>{{ card.title }}</h4>
                       <!-- PLACEHOLDER: Drag-and-drop cards with AlpineJS -->
                   </div>
               {% endfor %}
           </div>
       {% endfor %}
   </div>
   {% endblock %}
   ```

### Forms

1. **Card Update Form:**

   `templates/boards/card_form.html`:

   ```html
   {% extends "base.html" %}
   {% block title %}Edit Card{% endblock %}
   {% block content %}
   <form method="post">
       {% csrf_token %}
       {{ form.as_p }}
       <button type="submit">Update</button>
   </form>
   {% endblock %}
   ```

---

## 4. HTMX and AlpineJS Integration

### Real-time Card Updates Using HTMX

1. **Board List Page:**

   ```html
   <div hx-get="/dashboard/" hx-trigger="every 10s" hx-target="#board-list">
       <!-- Board list content -->
   </div>
   ```

### Drag-and-Drop Cards Using AlpineJS

1. **AlpineJS Setup:**

   Add to `board_detail.html`:

   ```html
   <div x-data="dragAndDrop()" x-init="initDragAndDrop()">
       <!-- Add cards here and implement drag-and-drop logic -->
   </div>
   ```

2. **AlpineJS Example:**

   ```javascript
   function dragAndDrop() {
     return {
       initDragAndDrop() {
         // Initialize drag-and-drop functionality
       }
     };
   }
   ```

---

## 5. API Integrations

*(No external API integrations in this tutorial)*

---

## 6. Testing Considerations

1. **Model Tests:**

   `boards/tests.py`:

   ```python
   from django.test import TestCase
   from .models import Board, List, Card

   class BoardModelTests(TestCase):
       def test_board_creation(self):
           board = Board.objects.create(name="Test Board")
           self.assertEqual(str(board), "Test Board")
   ```

2. **View Tests:**

   `boards/tests.py`:

   ```python
   from django.test import Client, TestCase
   from django.urls import reverse

   class BoardViewTests(TestCase):
       def test_board_list_view(self):
           response = self.client.get(reverse('board_list'))
           self.assertEqual(response.status_code, 200)
           self.assertContains(response, "Boards")
   ```

---

## 7. Deployment Notes

1. **Ensure all static files are collected:**

   ```bash
   python manage.py collectstatic
   ```

2. **Consider using a persistent storage backend for media files (e.g., AWS S3).**

3. **Configure the app to use a production-ready database like PostgreSQL.**

4. **Configure a suitable web server (such as Gunicorn) and reverse proxy (such as Nginx).**

This structured guide provides a comprehensive backend setup and outlines frontend integration points, ensuring a solid foundation for further frontend expansion.

---

Frontend
Given the scope of the task, I'll provide a comprehensive breakdown of each aspect of the frontend implementation for the "Kanban" application. Below are the code snippets and explanations for each step:

### 1. HTML Structure

#### Base Template (`templates/base.html`)

The base template will include a navigation bar, main content block, and footer. Additional scripts will be added at the bottom.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kanban - {% block title %}{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/styles.css' %}">
    <script defer src="https://unpkg.com/alpinejs@3" defer></script>
</head>
<body>
    <nav>
        <ul>
            <li><a href="{% url 'board_list' %}">Dashboard</a></li>
            <li><a href="{% url 'profile' %}">Profile</a></li>
            <li><a href="{% url 'settings' %}">Settings</a></li>
            <li><a href="{% url 'logout' %}">Logout</a></li>
        </ul>
    </nav>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer>
        <p>&copy; 2023 Kanban App</p>
    </footer>
    {% block scripts %}{% endblock %}
</body>
</html>
```

#### Dashboard Page (`templates/boards/board_list.html`)

```html
{% extends "base.html" %}
{% block title %}Dashboard{% endblock %}
{% block content %}
<div>
    <button id="createBoardButton">Create New Board</button>
    <div id="boardList">
        {% for board in boards %}
            <div class="board">
                <h2><a href="{% url 'board_detail' board.id %}">{{ board.name }}</a></h2>
            </div>
        {% endfor %}
    </div>
</div>
{% endblock %}
```

#### Board Detail Page (`templates/boards/board_detail.html`)

```html
{% extends "base.html" %}
{% block title %}{{ board.name }}{% endblock %}
{% block content %}
<div>
    <button id="addListButton">Add List</button>
    <div class="lists-container" x-data="dragAndDrop()" x-init="initDragAndDrop()">
        {% for list in board.lists.all %}
            <div class="list" x-data="{ open: false }">
                <h3 @click="open = !open" :aria-expanded="open">{{ list.name }}</h3>
                <div class="cards" x-show="open">
                    {% for card in list.cards.all %}
                        <div class="card" draggable="true" x-on:dragstart="dragStart($event)" x-on:drop="drop($event)" x-on:dragover="dragOver($event)">
                            <h4>{{ card.title }}</h4>
                            <p>{{ card.description }}</p>
                        </div>
                    {% endfor %}
                </div>
            </div>
        {% endfor %}
    </div>
</div>
{% endblock %}
{% block scripts %}
<script src="{% static 'js/dragAndDrop.js' %}"></script>
{% endblock %}
```

#### Card Update Form (`templates/boards/card_form.html`)

```html
{% extends "base.html" %}
{% block title %}Edit Card{% endblock %}
{% block content %}
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Update</button>
</form>
{% endblock %}
```

### 2. CSS Styling (`static/css/styles.css`)

```css
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f9;
    color: #333;
    margin: 0;
    padding: 0;
}

nav {
    background: #333;
    color: #fff;
    padding: 10px;
    text-align: center;
}

nav ul {
    list-style-type: none;
}

nav ul li {
    display: inline;
    margin: 0 15px;
}

nav ul li a {
    color: #fff;
    text-decoration: none;
}

main {
    padding: 20px;
}

footer {
    background: #333;
    color: #fff;
    text-align: center;
    padding: 10px;
    position: fixed;
    bottom: 0;
    width: 100%;
}

.board, .list, .card {
    background: #fff;
    padding: 10px;
    margin: 10px 0;
    border-radius: 5px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.cards {
    padding-left: 20px;
}
```

### 3. JavaScript and Interactivity

#### HTMX for Real-Time Updates

In the `board_detail.html` for the board lists:

```html
<div id="boardList" hx-get="/dashboard/" hx-trigger="every 10s"
     hx-target="#boardList">
    <!-- Board List will be updated dynamically -->
</div>
```

#### AlpineJS for Drag-and-Drop (`static/js/dragAndDrop.js`)

```javascript
function dragAndDrop() {
    return {
        dragStart(event) {
            event.dataTransfer.setData('text', event.target.id);
        },
        drop(event) {
            event.preventDefault();
            const data = event.dataTransfer.getData('text');
            const card = document.getElementById(data);
            event.target.appendChild(card);
        },
        dragOver(event) {
            event.preventDefault();
        },
        initDragAndDrop() {
            document.querySelectorAll('.card').forEach(card => {
                card.id = `card-${Math.random()}`;
            });
        },
    };
}
```

### 4. Forms

Ensure forms are styled properly in `styles.css` and include client-side validation with HTML5 validation attributes.

### 5. Components

Create reusable components using Django template inheritance and include them in relevant templates.

### 6. Navigation and User Flow

The navigation bar is included in the base template to make navigation consistent across pages. This improves user experience.

### 7. Error Handling and User Feedback

Add messages to the base template for success or error notifications using Django's messaging framework:

```html
{% if messages %}
<div class="messages">
  {% for message in messages %}
    <div class="message {{ message.tags }}">
      {{ message }}
    </div>
  {% endfor %}
</div>
{% endif %}
```

### 8. Accessibility

Ensure semantic HTML by using appropriate elements like button, h1-h6 for headings, and understanding roles with ARIA attributes when necessary (e.g., `:aria-expanded="open"`).

This implementation focuses on maintaining a clean separation of concerns, using Django's templating for dynamic content while incorporating HTMX for real-time functionality and AlpineJS for interactive features like drag-and-drop. All CSS and scripts are organized systematically to ensure maintainability and scalability of the application.
    
---
## QUIZ
## What is the purpose of the 'related_name' attribute in Django models?

The `related_name` attribute in Django models is used within a `ForeignKey` or `ManyToManyField` to specify the name of the reverse relation from the related model back to the model where the field is defined. It helps in accessing related objects conveniently.

**Example:**

```python
class List(models.Model):
    board = models.ForeignKey(Board, related_name='lists', on_delete=models.CASCADE)

class Board(models.Model):
    # ... other fields ...
```

In this example, `related_name='lists'` allows us to access all lists associated with a specific board instance using `board.lists.all()`. 

**Without `related_name`:** You would have to use the default reverse manager like `board.list_set.all()`, which is less descriptive.

## What are Django Class-Based Views (CBVs) and why are they used here?

Class-Based Views (CBVs) in Django provide a way to structure views by inheriting pre-defined generic view classes. They promote code reusability and organization by encapsulating common view logic.

**Example:**

```python
class BoardListView(ListView):
    model = Board
    template_name = 'boards/board_list.html'
    context_object_name = 'boards'
```

Here, `BoardListView` inherits from Django's generic `ListView` CBV, handling common logic for displaying a list of objects. We customize it by specifying the model (`Board`), the template (`board_list.html`), and the context variable name (`boards`).

**Benefits:**

- **Reusability:** Leverage built-in functionality for common view patterns.
- **Organization:** Structure code logically, improving readability and maintainability.

## How does HTMX update the board list in real-time?

HTMX enables real-time updates by making partial page updates instead of full page reloads. In the `board_detail.html` snippet:

```html
<div id="boardList" hx-get="/dashboard/" hx-trigger="every 10s"
     hx-target="#boardList">
    <!-- Board List will be updated dynamically -->
</div>
```

- `hx-get="/dashboard/"`: Instructs HTMX to make a GET request to the `/dashboard/` URL.
- `hx-trigger="every 10s"`: Specifies the trigger for the request, in this case, every 10 seconds.
- `hx-target="#boardList"`: Identifies the target element to be updated with the response from the server.

Every 10 seconds, HTMX fetches the latest content from `/dashboard/` and updates the `#boardList` element, providing a seamless real-time update experience.

## Explain the role of AlpineJS in the drag-and-drop functionality.

AlpineJS is a lightweight JavaScript framework used here to implement the drag-and-drop interaction for the cards.

**Key points:**

- `x-data="dragAndDrop()"`: Initializes the AlpineJS component and associates it with the `dragAndDrop` function.
- `x-on:dragstart/drop/dragover`: These directives listen for drag-and-drop events and call the corresponding functions defined in the `dragAndDrop` object.
- The `dragStart`, `drop`, and `dragOver` functions within the `dragAndDrop` object handle the logic for initiating the drag, dropping the element, and managing the drag-over state, respectively.

AlpineJS simplifies DOM manipulation and event handling, making the drag-and-drop implementation more concise and maintainable.

## How does the '{% csrf_token %}' tag work in Django templates?

The `{% csrf_token %}` tag is crucial for protecting against Cross-Site Request Forgery (CSRF) attacks in Django. When rendered, it generates a hidden input field within the form that contains a unique, unpredictable token.

**Functionality:**

1. On page load, the server generates a CSRF token and includes it in the template as a hidden field.
2. When the form is submitted, Django checks if the token from the client matches the token stored on the server.
3. If the tokens match, the request is considered safe; if not, it's rejected as a potential CSRF attack.

This mechanism ensures that only requests originating from the genuine website can modify data, preventing malicious actors from exploiting user sessions.

## Why is it recommended to use a production-ready database like PostgreSQL instead of the default SQLite for deployment?

While SQLite, the default Django database, is suitable for development and testing, it's not recommended for production environments due to limitations in:

- **Concurrency:** SQLite handles one write operation at a time, potentially leading to bottlenecks with concurrent users.
- **Scalability:** SQLite is file-based and might not perform well with large datasets or high traffic.

**PostgreSQL Advantages:**

- **Robustness:** Handles concurrent requests efficiently and offers better data integrity.
- **Scalability:** Designed to handle large databases and high traffic loads.
- **Features:** Provides advanced features like transaction integrity, replication, and extensions.

For a production-ready application, using PostgreSQL ensures better performance, reliability, and scalability compared to SQLite.
