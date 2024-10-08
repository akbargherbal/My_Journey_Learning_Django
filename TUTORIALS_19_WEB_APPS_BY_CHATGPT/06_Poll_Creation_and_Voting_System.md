

# Poll Creation and Voting System
## An app for creating polls and allowing users to vote, with data visualization and prevention of multiple votes.

---

# Backend
## Tutorial for Building a Poll Creation and Voting System using Django

This tutorial will guide you through the backend implementation using Django and will provide a structured outline for the frontend components. The approach covers creating and voting on polls, viewing results, and enforcing a one-vote-per-user constraint.

---

### 1. Project Setup

#### Django Project and App Creation

1. **Set Up Django Environment:**
   ```bash
   pip install django
   django-admin startproject pollsystem
   cd pollsystem
   ```

2. **Create Django Apps:**
   ```bash
   python manage.py startapp polls
   python manage.py startapp users
   ```

#### Initial Settings Configuration

1. **Configure `settings.py`:**

   - Add `polls` and `users` to `INSTALLED_APPS`.
   ```python
   INSTALLED_APPS = [
       ...
       'polls',
       'users',
   ]
   ```

2. **Database Configuration:**
   Ensure the database settings point to your DB (using default SQLite here).

---

### 2. Backend Implementation

#### Models

**Polls App Models**

- **Poll Model:**
  ```python
  from django.db import models
  
  class Poll(models.Model):
      title = models.CharField(max_length=255)
      start_date = models.DateTimeField(auto_now_add=True)
      end_date = models.DateTimeField()
  ```

- **Question Model:**
  ```python
  class Question(models.Model):
      poll = models.ForeignKey(Poll, related_name='questions', on_delete=models.CASCADE)
      text = models.TextField()
  ```

- **Choice Model:**
  ```python
  class Choice(models.Model):
      question = models.ForeignKey(Question, related_name='choices', on_delete=models.CASCADE)
      choice_text = models.CharField(max_length=255)
  ```

- **Vote Model:**
  ```python
  from django.contrib.auth.models import User

  class Vote(models.Model):
      user = models.ForeignKey(User, on_delete=models.CASCADE)
      choice = models.ForeignKey(Choice, on_delete=models.CASCADE)
      timestamp = models.DateTimeField(auto_now_add=True)

      class Meta:
          unique_together = ('user', 'choice')
  ```

**Users App Models**

- **UserProfile Model:**
  ```python
  from django.contrib.auth.models import User

  class UserProfile(models.Model):
      user = models.OneToOneField(User, on_delete=models.CASCADE)
      bio = models.TextField(blank=True)
  ```

#### Views

**Polls App Views**

- **PollListView:**
  ```python
  from django.views.generic import ListView
  from .models import Poll

  class PollListView(ListView):
      model = Poll
      template_name = 'polls/list.html'
  ```

- **PollDetailView:**
  ```python
  from django.views.generic import DetailView

  class PollDetailView(DetailView):
      model = Poll
      template_name = 'polls/detail.html'
  ```

- **VoteView:**
  ```python
  from django.views import View
  from django.shortcuts import get_object_or_404, redirect
  from .models import Choice, Vote

  class VoteView(View):
      def post(self, request, poll_id, question_id):
          choice_id = request.POST.get('choice')
          choice = get_object_or_404(Choice, id=choice_id)
          vote, created = Vote.objects.get_or_create(user=request.user, choice=choice)
          return redirect('polls:results', poll_id=poll_id)
  ```

- **ResultsView:**
  ```python
  from django.views.generic import TemplateView

  class ResultsView(TemplateView):
      template_name = 'polls/results.html'

      def get_context_data(self, **kwargs):
          context = super().get_context_data(**kwargs)
          # Add results calculation logic here
          return context
  ```

**Users App Views**

- **ProfileView and LoginView:** Utilize Django's built-in authentication views or customize as needed.

#### URL Configurations

1. **Polls URLs (`polls/urls.py`):**
   ```python
   from django.urls import path
   from .views import PollListView, PollDetailView, VoteView, ResultsView

   app_name = 'polls'

   urlpatterns = [
       path('', PollListView.as_view(), name='list'),
       path('<int:pk>/', PollDetailView.as_view(), name='detail'),
       path('<int:poll_id>/vote/', VoteView.as_view(), name='vote'),
       path('<int:poll_id>/results/', ResultsView.as_view(), name='results'),
   ]
   ```

2. **Users URLs (`users/urls.py`):**
   Configure profile and login views.

---

### 3. Frontend Structure

#### Base Template Outline

- **Base Template (`base.html`):**
  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>{% block title %}Poll System{% endblock %}</title>
      {% block styles %}{% endblock %}
  </head>
  <body>
      <header>
          <nav>
              <!-- Placeholder for navigation -->
          </nav>
      </header>

      <main>
          {% block content %}{% endblock %}
      </main>

      <footer>
          <!-- Placeholder for footer -->
      </footer>

      {% block scripts %}{% endblock %}
  </body>
  </html>
  ```

#### Page-Specific Templates

**Home Page Template (`list.html`):**

- Poll List and Login Button
  ```html
  {% extends 'base.html' %}
  {% block content %}
  <h1>Available Polls</h1>
  <div id="poll-list">
      <div hx-get="/polls/" hx-trigger="scroll from-bottom">
          <!-- PLACEHOLDER: Poll items -->
      </div>
  </div>
  <a href="{% url 'users:login' %}">Login</a>
  {% endblock %}
  ```

**Poll Detail Page Template (`detail.html`):**

- Poll Question, Choices List, Vote Button
  ```html
  {% extends 'base.html' %}
  {% block content %}
  <h1>{{ poll.title }}</h1>
  {% for question in poll.questions.all %}
      <h2>{{ question.text }}</h2>
      <form hx-post="{% url 'polls:vote' poll.id question.id %}">
          {% for choice in question.choices.all %}
              <div x-data="{selected: false}">
                  <input type="radio" name="choice" value="{{ choice.id }}" x-model="selected">
                  {{ choice.choice_text }}
              </div>
          {% endfor %}
          <button type="submit">Vote</button>
      </form>
  {% endfor %}
  {% endblock %}
  ```

**Results Page Template (`results.html`):**

- Results Chart
  ```html
  {% extends 'base.html' %}
  {% block content %}
  <h1>Poll Results</h1>
  <div x-data="{ chartData: [] }">
      <!-- Results chart goes here -->
  </div>
  <a href="{% url 'polls:list' %}">Back to Polls</a>
  {% endblock %}
  ```

#### Forms

- **Django forms (`forms.py`):** Not needed if using basic inline forms.

---

### 4. HTMX and AlpineJS Integration

**HTMX Features**

- **LoadMorePolls:**
  ```html
  <div hx-get="/polls/" hx-trigger="revealed" hx-target="#poll-list">
      <!-- Poll items -->
  </div>
  ```

- **VoteForm:**
  ```html
  <form hx-post="/polls/{poll_id}/vote/" hx-swap="outerHTML">
      <!-- Voting form -->
  </form>
  ```

**AlpineJS Usage**

- **Poll Interaction:**

  ```html
  <div x-data="{ selected: null }">
      <!-- Manage choice selection toggle -->
  </div>
  ```

- **Results Display:**

  ```html
  <div x-data="{ results: [] }" x-init="fetchResults()">
      <!-- Animate results display changes -->
  </div>
  ```

---

### 5. API Integrations

- **Fetch Poll Data (If applicable):** Generally not needed without external API.

---

### 6. Testing Considerations

- **Model Tests:**
  ```python
  from django.test import TestCase
  from .models import Poll, Question, Choice

  class PollModelTests(TestCase):
      def test_poll_creation(self):
          # Test logic here
          self.assertTrue(isinstance(poll, Poll))
  ```

- **View Tests:** Verify views are returning correct templates and data.

---

### 7. Deployment Notes

- Ensure all environment-specific settings (e.g., `DEBUG`, `ALLOWED_HOSTS`) are configured in `settings.py`.

- Use tools like Gunicorn and Nginx for deployment in production environments.

- Ensure secure handling of static and media files and consider using cloud storage for scalability.

---

By following this tutorial, you will have a solid backend implemented in Django and a structured outline for your frontend development. Integrate HTMX and AlpineJS for dynamic user interactions, ensuring modern UI responsiveness with minimal effort.

---

Frontend
To complete the frontend implementation for the Poll Creation and Voting System using Django, HTMX, and AlpineJS, I'll provide a comprehensive solution covering all aspects mentioned:

### 1. HTML Structure

#### Base Template (`base.html`)

The base template provides the fundamental structure, shared styles, scripts, and navigation.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Poll System{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/styles.css' %}">
    <script src="https://unpkg.com/htmx.org@1.6.1"></script>
    <script src="https://unpkg.com/alpinejs@3.10.2/dist/cdn.min.js" defer></script>
    {% block styles %}{% endblock %}
</head>
<body>
    <header>
        <nav>
            <a href="{% url 'polls:list' %}">Home</a>
            {% if user.is_authenticated %}
                <a href="{% url 'users:profile' %}">Profile</a>
                <a href="{% url 'logout' %}">Logout</a>
            {% else %}
                <a href="{% url 'login' %}">Login</a>
            {% endif %}
        </nav>
    </header>

    <main>
        {% block content %}{% endblock %}
    </main>

    <footer>
        <p>&copy; 2023 Poll System</p>
    </footer>

    {% block scripts %}{% endblock %}
</body>
</html>
```

#### Home Page (`list.html`)

The home page lists polls with dynamic loading using HTMX.

```html
{% extends 'base.html' %}
{% block title %}Available Polls{% endblock %}
{% block content %}
<h1>Available Polls</h1>
<div id="poll-list" hx-get="{% url 'polls:list' %}" hx-trigger="revealed" hx-target="#poll-list" hx-swap="innerHTML">
    <!-- Poll items will auto-load here -->
</div>
{% endblock %}
```

#### Poll Detail Page (`detail.html`)

Displays a poll with its questions and allows voting.

```html
{% extends 'base.html' %}
{% block title %}{{ poll.title }}{% endblock %}
{% block content %}
<h1>{{ poll.title }}</h1>
{% for question in poll.questions.all %}
    <h2>{{ question.text }}</h2>
    <form hx-post="{% url 'polls:vote' poll.id question.id %}" hx-swap="outerHTML">
        {% for choice in question.choices.all %}
            <div x-data="{ checked: false }">
                <label class="choice">
                    <input type="radio" name="choice" value="{{ choice.id }}" x-model="checked">
                    {{ choice.choice_text }}
                </label>
            </div>
        {% endfor %}
        <button type="submit">Vote</button>
    </form>
{% endfor %}
{% endblock %}
```

#### Results Page (`results.html`)

Shows poll results with basic visualization.

```html
{% extends 'base.html' %}
{% block title %}Results for {{ poll.title }}{% endblock %}
{% block content %}
<h1>Results for {{ poll.title }}</h1>
{% for question in poll.questions.all %}
    <h2>{{ question.text }}</h2>
    <div x-data="resultsData()">
        <template x-for="choice in {{ question.choices.all|json_script:'choices_'|safe }}">
            <div class="result-bar" :style="{ width: choice.percentage + '%' }">
                <span x-text="choice.text + ' - ' + choice.percentage + '%'" class="label"></span>
            </div>
        </template>
    </div>
{% endfor %}
<a href="{% url 'polls:list' %}">Back to Polls</a>
{% endblock %}
```

### 2. CSS Styling

Define styles in `static/css/styles.css`.

```css
body {
    font-family: Arial, sans-serif;
    line-height: 1.6;
}

header, footer {
    background-color: #333;
    color: #fff;
    padding: 10px 0;
    text-align: center;
}

nav a {
    color: #fff;
    margin: 0 10px;
    text-decoration: none;
}

h1, h2 {
    margin-top: 20px;
}

.choice {
    margin-bottom: 10px;
}

.result-bar {
    background-color: #4CAF50;
    color: white;
    padding: 5px;
    margin-bottom: 10px;
    border-radius: 5px;
}
```

### 3. JavaScript and Interactivity

#### HTMX Functionality

Here, HTMX is used to dynamically load content and handle voting:

- Add the poll list dynamically:
  ```html
  <div hx-get="{% url 'polls:list' %}" hx-trigger="revealed" hx-target="#poll-list">
      <!-- Poll items -->
  </div>
  ```

- Handle voting form submission:
  ```html
  <form hx-post="{% url 'polls:vote' poll_id question_id %}" hx-swap="outerHTML">
      <!-- Voting form -->
  </form>
  ```

#### AlpineJS Implementations

For interaction in the results display:

```javascript
function resultsData() {
    return {
        choices: [],
        init() {
            // Example of fetching and setting results data
            this.choices = JSON.parse(document.getElementById('choices_{{ question.id }}').textContent);
            this.choices.forEach(choice => {
                choice.percentage = (choice.votes / choice.totalVotes) * 100;
            });
        }
    }
}
```

### 4. Forms

Forms for voting are defined within the detail page template and leverage HTMX for interaction. Ensure to have the necessary form fields set in your Django form configurations or directly in the HTML if kept simple.

### 5. Components

For reusable components like choice items or result bars, define them in templates and use HTMX or Alpine for dynamic interaction:

- Poll choice:
  ```html
  <div x-data="{ checked: false }">
      <input type="radio" name="choice" value="{{ choice.id }}" x-model="checked" @change="console.log($event.target.value)">
      {{ choice.choice_text }}
  </div>
  ```

### 6. Navigation and User Flow

The navigation bar adapts based on user authentication, providing links for login/logout and profile management in the `base.html` template.

### 7. Error Handling and User Feedback

Implement basic feedback by enhancing form interactions in HTMX, direct feedback when AJAX calls are made:

```html
<button type="submit" hx-include="[name=choice]" hx-indicator="#loading" >Vote</button>
```

And provide a loading indicator:

```html
<div id="loading" style="display:none;">Loading...</div>
```

### 8. Accessibility

Ensure semantic HTML (using `<nav>`, `<header>`, and `<main>`), label inputs, and use ARIA attributes if necessary:

```html
<label for="choice_{{ choice.id }}">{{ choice.choice_text }}</label>
<input type="radio" id="choice_{{ choice.id }}" name="choice" value="{{ choice.id }}" aria-checked="false">
```

This setup provides a solid foundation for your poll system's frontend; feel free to expand with additional functionality like data visualization libraries for advanced charts, further responsiveness tweaks, and so on.
    
---
## QUIZ
## What is the purpose of `related_name` in Django models?

The `related_name` argument in a Django `ForeignKey` field is used to specify the reverse relationship from the related model back to the model where the `ForeignKey` is defined. In simpler terms, it creates a convenient way to access and query related objects from the other side of the relationship.

**Example:**

```python
class Question(models.Model):
    poll = models.ForeignKey(Poll, related_name='questions', on_delete=models.CASCADE)

class Poll(models.Model):
    # ... other fields

# Accessing questions related to a poll
poll = Poll.objects.get(pk=1)
questions = poll.questions.all() 
```

Without `related_name`, you would have to use the less convenient `_set` notation (e.g., `poll.question_set.all()`).

**In our tutorial**, `related_name` is used in models like `Question` (`related_name='questions'`) and `Choice` (`related_name='choices'`) to easily access related questions from a poll and related choices from a question, respectively.

## What does `auto_now_add=True` do in a Django model field?

In a Django model field, `auto_now_add=True` sets the field to the current datetime automatically when a new object is created. This value is then saved to the database.

**Example:**

```python
class Poll(models.Model):
    start_date = models.DateTimeField(auto_now_add=True)
```

When a new `Poll` object is created, `start_date` will automatically be set to the current date and time.

**In the tutorial**, this is used in the `Poll` model for `start_date` to keep track of when a poll was created.

## What are `{% block %}` tags in Django templates?

In Django templates, `{% block %}` tags define sections that can be extended and overridden by child templates. They provide a way to create reusable template components and customize the layout of your web pages.

**Example:**
```html
// base.html
<header>
    {% block header %}{% endblock %}
</header>

// child.html
{% extends 'base.html' %}
{% block header %}
    <h1>My Website</h1>
{% endblock %}
```

In this example, the `child.html` template extends `base.html` and overrides the `header` block to display a custom header.

**The tutorial** uses `{% block %}` tags extensively to structure the HTML layout. For instance, `base.html` defines blocks like `content` and `scripts` that are overridden by specific page templates like `list.html`, `detail.html`, and `results.html`.

## What does `hx-get` do in HTMX?

In HTMX, `hx-get` is an attribute used to make AJAX requests to the server and dynamically update parts of a webpage without full page reloads. It specifies the URL to send a GET request to.

**Example:**

```html
<div hx-get="/polls/data/" hx-trigger="click">
    Click to load data
</div>
```

When the div is clicked, HTMX sends a GET request to `/polls/data/` and updates the div's content with the response.

**The tutorial** uses `hx-get` to implement features like loading more polls dynamically as the user scrolls down the list on the home page.

## What does `hx-swap` do in HTMX?

`hx-swap` in HTMX determines how the response from an AJAX request should be swapped into the DOM. It specifies what part of the current page content should be replaced with the server's response.

**Example:**

```html
<button hx-post="/update/" hx-swap="innerHTML">
    Update Content
</button>
```

When the button is clicked, it sends a POST request to `/update/`, and the response from the server will replace the inner HTML content of the button itself.

**The tutorial** utilizes `hx-swap` to update the poll list dynamically with new polls fetched from the server, replacing the previous content within the designated container.

## What is the purpose of `x-data` in AlpineJS?

In AlpineJS, `x-data` is used to initialize a component's data and behavior. It defines a JavaScript object that acts as the component's data context, containing properties and methods that can be accessed and manipulated within the component's HTML.

**Example:**

```html
<div x-data="{ count: 0 }">
    <button x-on:click="count++">Increment</button>
    <span x-text="count"></span>
</div>
```

Here, `x-data` initializes `count` to 0. Clicking the button increments `count`, and the updated value is reflected in the span.

**The tutorial** uses `x-data` to manage the state of UI elements, like whether a choice is selected in a poll. For instance, it's used to toggle the `checked` state of radio buttons in the poll detail view.

## What does `x-model` do in AlpineJS?

In AlpineJS, `x-model` creates a two-way data binding between an input element and a property in the component's data context defined by `x-data`. This means that any changes to the input value automatically update the data property, and any changes to the data property are reflected back in the input value.

**Example:**

```html
<div x-data="{ name: 'John' }">
    <input type="text" x-model="name">
    <p>Hello, <span x-text="name"></span></p>
</div>
```

In this example, the input field's value is bound to the `name` property. Typing in the input updates the `name` property, which is immediately reflected in the paragraph below.

**The tutorial** utilizes `x-model` for real-time interactivity. When a user selects a choice in a poll on the detail page, `x-model` ensures the selected choice is reflected in the component's data, ready to be submitted with the form.

## What is the purpose of `json_script` in the Django template?

The `json_script` filter in Django templates is used to safely embed data as a JavaScript object within a `<script>` tag. This is particularly useful for passing data from the backend to JavaScript code in the frontend.

**Example:**

```python
# views.py
def my_view(request):
    context = {'my_data': {'name': 'Alice', 'age': 30}}
    return render(request, 'my_template.html', context)


# my_template.html
<script id="my-data" type="application/json">
    {{ my_data|json_script }}
</script>

<script>
    const myData = JSON.parse(document.getElementById('my-data').textContent);
    console.log(myData.name); // Outputs: "Alice"
</script>
```

**In our tutorial**, `json_script` is likely used to pass poll data from the Django backend to the frontend, where it might be processed by JavaScript to render charts, display results dynamically, or facilitate other interactive features.
