
# Guestbook
## An application allowing visitors to leave comments or messages. Includes spam prevention mechanisms.

---

# Backend
# Guestbook Application Tutorial

This tutorial will guide you through creating a "Guestbook" application using Django, focusing on backend implementation and a structured outline for frontend components.

## 1. Project Setup

### Django Project and App Creation

Start by setting up a new Django project and creating the required applications:

```bash
django-admin startproject guestbook_project
cd guestbook_project
python manage.py startapp guestbook
python manage.py startapp moderation
```

### Initial Settings Configuration

In `guestbook_project/settings.py`, add the new apps to the `INSTALLED_APPS` list:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'guestbook',
    'moderation',
]
```

Ensure to set up the databases and other middleware as needed. Configure `STATIC_URL`, `MEDIA_URL`, and other standard Django settings.

## 2. Backend Implementation

### Models

#### Guestbook App Models

Create models in `guestbook/models.py`:

```python
from django.db import models


class Entry(models.Model):
    visitor_name = models.CharField(max_length=100)
    message = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    is_spam = models.BooleanField(default=False)

    def __str__(self):
        return f"{self.visitor_name}: {self.message[:20]}"


class Visitor(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)

    def __str__(self):
        return self.name
```

#### Moderation App Models

Create models in `moderation/models.py`:

```python
class Review(models.Model):
    entry = models.ForeignKey('guestbook.Entry', on_delete=models.CASCADE)
    reviewed_at = models.DateTimeField(auto_now_add=True)
    approved = models.BooleanField(default=False)

    def __str__(self):
        return f"Review for {self.entry}"


class SpamFilter(models.Model):
    keyword = models.CharField(max_length=100)

    def __str__(self):
        return self.keyword
```

### Views

#### Guestbook Views

Update `guestbook/views.py`:

```python
from django.views.generic import ListView, CreateView
from .models import Entry
from .forms import EntryForm


class EntryListView(ListView):
    model = Entry
    template_name = 'guestbook/entry_list.html'
    context_object_name = 'entries'


class EntryCreateView(CreateView):
    model = Entry
    form_class = EntryForm
    template_name = 'guestbook/entry_form.html'
    success_url = '/'
```

#### Moderation Views

Update `moderation/views.py`:

```python
from django.views.generic import ListView, FormView
from .models import Review, SpamFilter
from .forms import SpamFilterForm


class ReviewListView(ListView):
    model = Review
    template_name = 'moderation/review_list.html'
    context_object_name = 'reviews'


class SpamFilterView(FormView):
    form_class = SpamFilterForm
    template_name = 'moderation/spam_filter.html'
    success_url = '/moderation/review/'

    def form_valid(self, form):
        form.save()
        return super().form_valid(form)
```

### URL Configurations

Add URL patterns in `guestbook/urls.py`:

```python
from django.urls import path
from .views import EntryListView, EntryCreateView

urlpatterns = [
    path('', EntryListView.as_view(), name='entry_list'),
    path('entries/new/', EntryCreateView.as_view(), name='entry_create'),
]
```

Add URL patterns in `moderation/urls.py`:

```python
from django.urls import path
from .views import ReviewListView, SpamFilterView

urlpatterns = [
    path('review/', ReviewListView.as_view(), name='review_list'),
    path('spam-filter/', SpamFilterView.as_view(), name='spam_filter'),
]
```

Include these in the project's main `urls.py`:

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('guestbook.urls')),
    path('moderation/', include('moderation.urls')),
]
```

## 3. Frontend Structure

### Base Template Outline

Create `templates/base.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Guestbook</title>
    <link rel="stylesheet" href="{% static 'styles.css' %}">
    {% block head %}{% endblock %}
</head>
<body>
    <nav>
        <!-- Placeholder for navigation -->
    </nav>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer>
        <!-- Placeholder for footer -->
    </footer>
    <script src="{% static 'scripts.js' %}"></script>
    {% block scripts %}{% endblock %}
</body>
</html>
```

### Page-Specific Templates

Create `guestbook/templates/guestbook/entry_list.html`:

```html
{% extends 'base.html' %}
{% block content %}
<h1>Guestbook Entries</h1>
<div id="entry-list">
    <!-- PLACEHOLDER: Iterate through `entries` and display them -->
</div>
<a href="{% url 'entry_create' %}">Leave a message</a>
{% endblock %}
```

Create `moderation/templates/moderation/review_list.html`:

```html
{% extends 'base.html' %}
{% block content %}
<h1>Review Entries</h1>
<ul>
    <!-- PLACEHOLDER: List `reviews` here -->
</ul>
{% endblock %}
```

### Forms

Create forms in `guestbook/forms.py`:

```python
from django import forms
from .models import Entry

class EntryForm(forms.ModelForm):
    class Meta:
        model = Entry
        fields = ['visitor_name', 'message']
```

Create forms in `moderation/forms.py`:

```python
from django import forms
from .models import SpamFilter

class SpamFilterForm(forms.ModelForm):
    class Meta:
        model = SpamFilter
        fields = ['keyword']
```

## 4. HTMX and AlpineJS Integration

### Interactive Features

- **LiveEntryUpdate**: Use HTMX to update entries live.
- **FormSubmission**: Seamless form submissions without reloading.

#### HTMX Example

In `entry_list.html`, setup HTMX for real-time updates:

```html
<div hx-get="/entries/live-update/" hx-trigger="load" hx-target="#entry-list">
    <!-- Entries will be dynamically loaded here -->
</div>
```

#### AlpineJS Example

In `spam_filter.html`, use AlpineJS for an interactive UI:

```html
<div x-data="{ keyword: '' }">
    <input x-model="keyword" type="text" placeholder="Enter spam keyword">
    <button @click="addKeyword()">Add Keyword</button>
</div>
```

## 5. API Integrations

This section can be expanded with specific API integration instructions as needed. For this tutorial, no external APIs are used.

## 6. Testing Considerations

### Key Test Cases

- **Model Tests**: Validate entry creation, modification, and spam detection.
- **View Tests**: Ensure pages render correctly and forms submit properly.
- **Spam Filter Logic**: Test the addition and application of spam filter keywords.

Example for model tests in `guestbook/tests.py`:

```python
from django.test import TestCase
from .models import Entry

class EntryModelTests(TestCase):
    def test_entry_creation(self):
        entry = Entry.objects.create(visitor_name='Jane Doe', message='Hello!')
        self.assertTrue(entry.created_at is not None)
```

## 7. Deployment Notes

- **Database Setup**: Ensure the production database is properly configured.
- **Static Files**: Use `collectstatic` to collect static files for deployment.
- **Security**: Use environment variables for sensitive settings like `SECRET_KEY`.

This tutorial provides a comprehensive guide to building the Guestbook application using Django, offering a solid foundation for backend implementation and guidance for structuring the frontend with integration points for HTMX and AlpineJS.

---

Frontend
To complete the frontend implementation for the Guestbook application, we'll address each required aspect step-by-step.

### 1. HTML Structure

#### Base Template (`templates/base.html`)
Here, we'll finalize the base structure and provide placeholders for navigation and content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Guestbook{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'styles.css' %}">
    {% block head %}{% endblock %}
</head>
<body>
    <nav>
        <ul>
            <li><a href="{% url 'entry_list' %}">Home</a></li>
            <li><a href="{% url 'entry_create' %}">New Entry</a></li>
            <li><a href="{% url 'review_list' %}">Moderation</a></li>
        </ul>
    </nav>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer>
        <p>Guestbook Application &copy; 2023</p>
    </footer>
    <script src="{% static 'htmx.min.js' %}"></script>
    <script src="{% static 'alpine.min.js' %}"></script>
    <script src="{% static 'scripts.js' %}"></script>
    {% block scripts %}{% endblock %}
</body>
</html>
```

#### Entry List Template (`guestbook/templates/guestbook/entry_list.html`)
Enhance the display of entries:

```html
{% extends 'base.html' %}
{% block title %}Guestbook Entries{% endblock %}
{% block content %}
<h1>Guestbook Entries</h1>
<div id="entry-list" hx-get="/entries/live-update/" hx-trigger="load">
    <ul>
        {% for entry in entries %}
        <li>
            <strong>{{ entry.visitor_name }}</strong>: {{ entry.message }}<br>
            <em>{{ entry.created_at|date:"F j, Y, g:i a" }}</em>
        </li>
        {% endfor %}
    </ul>
</div>
<a href="{% url 'entry_create' %}" class="btn">Leave a Message</a>
{% endblock %}
```

#### Entry Form Template (`guestbook/templates/guestbook/entry_form.html`)
Implement the form for creating entries:

```html
{% extends 'base.html' %}
{% block title %}New Entry{% endblock %}
{% block content %}
<h1>Create a New Entry</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit" class="btn">Submit</button>
</form>
{% endblock %}
```

#### Review List Template (`moderation/templates/moderation/review_list.html`)
Layout for reviewing entries:

```html
{% extends 'base.html' %}
{% block title %}Review Entries{% endblock %}
{% block content %}
<h1>Review Entries</h1>
<ul>
    {% for review in reviews %}
    <li>
        <strong>{{ review.entry.visitor_name }}</strong>: {{ review.entry.message }}<br>
        <em>Reviewed at {{ review.reviewed_at|date:"F j, Y, g:i a" }}</em><br>
        Approved: {{ review.approved }}
    </li>
    {% endfor %}
</ul>
{% endblock %}
```

#### Spam Filter Form (`moderation/templates/moderation/spam_filter.html`)
Interactive UI with AlpineJS:

```html
{% extends 'base.html' %}
{% block title %}Spam Filter{% endblock %}
{% block content %}
<h1>Add Spam Keyword</h1>
<form method="post" x-data="{ keyword: '' }">
    {% csrf_token %}
    <input type="text" name="keyword" x-model="keyword" placeholder="Enter spam keyword">
    <button type="submit" class="btn" @click.prevent="$dispatch('submit')">Add Keyword</button>
</form>
{% endblock %}
```

### 2. CSS Styling

Create `/static/styles.css`:

```css
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    line-height: 1.6;
    background-color: #f4f4f9;
}

nav ul {
    list-style: none;
    padding: 10px;
    background: #333;
    overflow: hidden;
}

nav ul li {
    display: inline;
    padding: 0 15px;
}

nav a {
    color: #fff;
    text-decoration: none;
}

main {
    padding: 20px;
}

h1 {
    color: #333;
}

.btn {
    padding: 10px 20px;
    background-color: #333;
    color: #fff;
    text-decoration: none;
    border-radius: 5px;
}

footer {
    background-color: #333;
    color: #fff;
    text-align: center;
    padding: 10px 0;
    position: absolute;
    bottom: 0;
    width: 100%;
}
```

### 3. JavaScript and Interactivity

#### Using HTMX for Live Updates
No additional code is required for HTMX as long as the server endpoint (`/entries/live-update/`) is handling AJAX requests and returning properly formatted HTML snippets.

#### AlpineJS for Spam Filtering
Already provided in the `spam_filter.html`. You may add methods in `scripts.js` if needed.

### 4. Forms

Done in the HTML sections with items like `{% csrf_token %}` and standard Django form rendering.

### 5. Components

Consider abstracting common parts such as `<nav>`, footers, and buttons into `{% include %}` templates for reuse across multiple pages:

Create `components/_navbar.html`:

```html
<nav>
    <ul>
        <li><a href="{% url 'entry_list' %}">Home</a></li>
        <li><a href="{% url 'entry_create' %}">New Entry</a></li>
        <li><a href="{% url 'review_list' %}">Moderation</a></li>
    </ul>
</nav>
```

Include it in `base.html`:

```html
{% include 'components/_navbar.html' %}
```

### 6. Navigation and User Flow

Using consistent navigation links in the navbar ensures logical user flow.

### 7. Error Handling and User Feedback

Add modals or alerts in `scripts.js`. For example:

```javascript
function showSuccessMessage(message) {
    alert(`Success: ${message}`);
}

function showErrorMessage(message) {
    alert(`Error: ${message}`);
}
```

Implement in forms:

```html
<form method="post" onsubmit="showSuccessMessage('Entry submitted successfully!')">
    <!-- Form content here -->
</form>
```

### 8. Accessibility

Ensure initial HTML uses semantic tags such as `<main>`, consider color contrast (as seen in CSS), and ensure all interactive elements are keyboard accessible. Use ARIA attributes if necessary, for example:

```html
<button aria-label="Submit form">Submit</button>
```

### Final Notes

These implementations integrate seamlessly with the Django backend as per the tutorial specifications. The setup caters to accessibility guidelines, has a user-friendly interface, and is structured for easy navigation. Make sure to test extensively both the frontend interactions and backend connections to ensure smooth operation.
    