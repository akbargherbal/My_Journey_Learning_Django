

# Movie Database and Review System
## An app to search and review movies, integrating with external APIs (e.g., IMDB) for movie data and user ratings.

---

# Backend
# Movie Database and Review System: Detailed Implementation Tutorial

This tutorial provides a comprehensive guide to building a Movie Database and Review System using Django. We will focus primarily on backend implementation and outline the structure for frontend components using Django Template Language, HTMX, and AlpineJS.

## 1. Project Setup

### Django Project and App Creation
1. **Install Django**:
   ```bash
   pip install django
   ```

2. **Create a Django Project**:
   ```bash
   django-admin startproject movie_database
   cd movie_database
   ```

3. **Create Django Apps**:
   ```bash
   python manage.py startapp movies
   python manage.py startapp reviews
   python manage.py startapp api_integration
   ```

### Initial Settings Configuration
- Open `movie_database/settings.py` and add the apps to `INSTALLED_APPS`:
  ```python
  INSTALLED_APPS = [
      # Other apps
      'movies',
      'reviews',
      'api_integration',
  ]
  ```

## 2. Backend Implementation

### Models

#### 2.1. Movies App Models
- **`models.py` in `movies` app**:
  ```python
  from django.db import models

  class Genre(models.Model):
      name = models.CharField(max_length=100)

  class Movie(models.Model):
      title = models.CharField(max_length=200)
      release_date = models.DateField()
      genres = models.ManyToManyField(Genre)
      imdb_id = models.CharField(max_length=10)

      def average_rating(self):
          return self.rating_set.aggregate(models.Avg('score'))['score__avg']

  class Rating(models.Model):
      movie = models.ForeignKey(Movie, on_delete=models.CASCADE)
      score = models.IntegerField(choices=[(i, str(i)) for i in range(1, 6)])
  ```

#### 2.2. Reviews App Models
- **`models.py` in `reviews` app**:
  ```python
  from django.db import models
  from django.contrib.auth.models import User
  from movies.models import Movie

  class Review(models.Model):
      movie = models.ForeignKey(Movie, on_delete=models.CASCADE)
      user = models.ForeignKey(User, on_delete=models.CASCADE)
      content = models.TextField()
  ```

### Views

#### 2.3. Movies App Views
- **`views.py` in `movies` app**:
  ```python
  from django.views.generic import ListView, DetailView
  from .models import Movie

  class MovieListView(ListView):
      model = Movie
      template_name = 'movies/movie_list.html'
      context_object_name = 'movies'

  class MovieDetailView(DetailView):
      model = Movie
      template_name = 'movies/movie_detail.html'
  ```

#### 2.4. Reviews App Views
- **`views.py` in `reviews` app**:
  ```python
  from django.views.generic.edit import CreateView, UpdateView
  from .models import Review

  class CreateReviewView(CreateView):
      model = Review
      template_name = 'reviews/create_review.html'
      fields = ['content']

      def form_valid(self, form):
          form.instance.user = self.request.user
          return super().form_valid(form)

  class EditReviewView(UpdateView):
      model = Review
      template_name = 'reviews/edit_review.html'
      fields = ['content']
  ```

#### 2.5. API Integration Views
- **`views.py` in `api_integration` app**:
  ```python
  from django.http import JsonResponse
  import requests

  def fetch_movie_data_view(request, imdb_id):
      response = requests.get(f'https://api.imdb.com/?id={imdb_id}')
      data = response.json()
      # Logic to save or update Movie and Genre instances
      return JsonResponse(data)
  ```

### URL Configurations

#### 2.6. URL Patterns
- **`urls.py` in each app**:
  - **Movies App**:
    ```python
    from django.urls import path
    from .views import MovieListView, MovieDetailView

    urlpatterns = [
        path('', MovieListView.as_view(), name='movie-list'),
        path('<int:pk>/', MovieDetailView.as_view(), name='movie-detail'),
    ]
    ```

  - **Reviews App**:
    ```python
    from django.urls import path
    from .views import CreateReviewView, EditReviewView

    urlpatterns = [
        path('', CreateReviewView.as_view(), name='create-review'),
        path('edit/<int:pk>/', EditReviewView.as_view(), name='edit-review'),
    ]
    ```
  
  - **API Integration App**:
    ```python
    from django.urls import path
    from .views import fetch_movie_data_view

    urlpatterns = [
        path('fetch-movie-data/', fetch_movie_data_view, name='fetch-movie-data'),
    ]
    ```

  - **Include in Main Project URLs**:
    Update the `urls.py` in the main project:
    ```python
    from django.urls import path, include

    urlpatterns = [
        path('movies/', include('movies.urls')),
        path('reviews/', include('reviews.urls')),
        path('api/', include('api_integration.urls')),
    ]
    ```

## 3. Frontend Structure

### Base Template Outline

- **`base.html` Template**:
  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>{% block title %}Movie Database{% endblock %}</title>
      {% block styles %}
      <!-- Include CSS here -->
      {% endblock %}
  </head>
  <body>
      <nav>
        <!-- PLACEHOLDER: Navbar -->
      </nav>

      <main>
          {% block content %}
          <!-- Content goes here -->
          {% endblock %}
      </main>

      <footer>
        <!-- PLACEHOLDER: Footer -->
      </footer>

      {% block scripts %}
      <!-- Include JS files here -->
      {% endblock %}
  </body>
  </html>
  ```

### Page-Specific Templates
- **HomePage** (`home.html`):
  ```html
  {% extends "base.html" %}
  {% block content %}
  <!-- PLACEHOLDER: SearchBar component -->
  <!-- PLACEHOLDER: TrendingMovies component -->
  {% endblock %}
  ```

- **MovieDetailPage** (`movie_detail.html`):
  ```html
  {% extends "base.html" %}
  {% block content %}
  <!-- PLACEHOLDER: MovieInfo component -->
  <!-- PLACEHOLDER: ReviewSection component -->
  {% endblock %}
  ```

- **UserReviewPage** (`create_review.html`):
  ```html
  {% extends "base.html" %}
  {% block content %}
  <!-- PLACEHOLDER: ReviewForm component -->
  {% endblock %}
  ```

### Forms

- **Django Form Definitions**:
  - Create forms in `reviews/forms.py`:
    ```python
    from django import forms
    from .models import Review

    class ReviewForm(forms.ModelForm):
        class Meta:
            model = Review
            fields = ['content']
    ```

- **Outline of Form Templates**:
  - `create_review.html`:
    ```html
    <form method="post">{% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Submit</button>
    </form>
    ```

## 4. HTMX and AlpineJS Integration

### HTMX Features

- **LiveSearch**:
  - Use HTMX to dynamically fetch and display search results:
  ```html
  <input hx-get="/movies/?q={search_query}" hx-trigger="keyup"
         hx-target="#search-results" placeholder="Search Movies...">
  <div id="search-results"></div>
  ```

- **SubmitReviewForm**:
  - Submit review form without reloading page:
  ```html
  <form hx-post="{% url 'create-review' %}" hx-target="#review-section">
      {{ form.as_p }}
      <button type="submit">Submit</button>
  </form>
  ```

### AlpineJS Usage

- **MovieSortDropdown**:
  - Dynamic sorting using AlpineJS:
  ```html
  <div x-data="{ sortOrder: 'desc' }">
      <button @click="sortOrder = 'asc'">Sort Ascending</button>
      <button @click="sortOrder = 'desc'">Sort Descending</button>
      <!-- Display sorted movies based on sortOrder -->
  </div>
  ```

- **ReviewAccordion**:
  - Toggle review details using AlpineJS:
  ```html
  <div x-data="{ open: false }">
      <button @click="open = !open">Toggle Review</button>
      <div x-show="open">
          <!-- Review details -->
      </div>
  </div>
  ```

## 5. API Integrations

### API Client Setup

- Set up API clients in `api_integration/models.py` or as a service:
  ```python
  import requests

  def fetch_movie_data(imdb_id):
      response = requests.get(f'https://api.imdb.com/?i={imdb_id}')
      if response.status_code == 200:
          data = response.json()
          # handle data
      else:
          # handle error
      return data
  ```

### View and Logic for API Interactions

- **`views.py` in `api_integration` app**:
  ```python
  def fetch_movie_data_view(request):
      imdb_id = request.GET.get('imdb_id', None)
      if imdb_id:
          data = fetch_movie_data(imdb_id)
          # Handle data, return response
          return JsonResponse(data)
      return JsonResponse({'error': 'No IMDb ID provided'}, status=400)
  ```

## 6. Testing Considerations

### Outline of Key Test Cases

- **Models**:
  ```python
  from django.test import TestCase
  from .models import Movie, Rating

  class MovieModelTests(TestCase):
      def test_average_rating(self):
          movie = Movie.objects.create(title="Test Movie")
          Rating.objects.create(movie=movie, score=4)
          Rating.objects.create(movie=movie, score=5)
          avg = movie.average_rating()
          self.assertEqual(avg, 4.5)
  ```

- **Views**:
  ```python
  from django.test import TestCase, Client
  from django.urls import reverse

  class MovieViewTests(TestCase):
      def test_movie_list_view(self):
          response = self.client.get(reverse('movie-list'))
          self.assertEqual(response.status_code, 200)
  ```

## 7. Deployment Notes

- **Database Setup**: Ensure the database is configured correctly with Django settings.
- **Static Files**: Collect static files using `python manage.py collectstatic` and serve them using a web server like Nginx or Apache.
- **Django Configuration**: Secure your Django settings (e.g., secret key, databases) for production using environment variables.
- **WSGI Setup**: Configure `gunicorn` or another WSGI server to serve your application.
- **Load Balancer**: Consider using a load balancer for distributing traffic and increasing reliability.

By following this tutorial, you will have a robust backend in Django with a clear structure for implementing frontend components using modern JavaScript libraries. This separation of concerns allows for efficient development and future enhancements to both the frontend and backend.

---

Frontend
To provide a comprehensive front-end implementation for the Movie Database and Review System, we'll focus on completing the HTML, CSS, and JavaScript aspects for the application outlined in the tutorial. This includes integrating HTMX and AlpineJS for interactivity and ensuring the layout is responsive and accessible.

### 1. HTML Structure

Here is the complete HTML for each template:

#### `base.html`
This template serves as a common structure that other templates will extend.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Movie Database{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/styles.css' %}">
    {% block styles %}
    <!-- Additional page-specific styles -->
    {% endblock %}
</head>
<body>
    <nav>
        <div class="container">
            <ul>
                <li><a href="{% url 'movie-list' %}">Home</a></li>
                <li><a href="{% url 'create-review' %}">Add Review</a></li>
                <li><a href="{% url 'fetch-movie-data' %}">Integrate Movie Data</a></li>
            </ul>
        </div>
    </nav>

    <main class="container">
        {% block content %}
        <!-- Page-specific content -->
        {% endblock %}
    </main>

    <footer>
        <div class="container">
            <p>&copy; 2023 Movie Database. All Rights Reserved.</p>
        </div>
    </footer>

    <script src="https://unpkg.com/htmx.org"></script>
    <script src="https://unpkg.com/alpinejs" defer></script>
    {% block scripts %}
    <!-- Additional page-specific scripts -->
    {% endblock %}
</body>
</html>
```

#### `movie_list.html`
Displays a list of movies.

```html
{% extends "base.html" %}
{% block title %}Movies - Movie Database{% endblock %}
{% block content %}
<h1>Movies</h1>
<div>
    <input type="text" id="search-bar" 
           hx-get="{% url 'movie-list' %}?q={query}" 
           hx-trigger="keyup changed delay:500ms"
           hx-target="#movies-list"
           placeholder="Search Movies...">
</div>
<div id="movies-list">
    {% for movie in movies %}
    <div class="movie-card">
        <h2><a href="{% url 'movie-detail' movie.pk %}">{{ movie.title }}</a></h2>
        <p>Release Date: {{ movie.release_date }}</p>
        <p>Genres: {% for genre in movie.genres.all %}{{ genre.name }}{% if not forloop.last %}, {% endif %}{% endfor %}</p>
        <p>Average Rating: {{ movie.average_rating|default:"N/A" }}</p>
    </div>
    {% empty %}
    <p>No movies found.</p>
    {% endfor %}
</div>
{% endblock %}
```

#### `movie_detail.html`
Shows details and reviews of a specific movie.

```html
{% extends "base.html" %}
{% block title %}{{ object.title }} - Movie Details{% endblock %}
{% block content %}
<h1>{{ object.title }}</h1>
<div class="movie-info">
    <p>Release Date: {{ object.release_date }}</p>
    <p>Genres: {% for genre in object.genres.all %}{{ genre.name }}{% if not forloop.last %}, {% endif %}{% endfor %}</p>
    <p>Average Rating: {{ object.average_rating|default:"N/A" }}</p>
</div>

<div x-data="{ open: false }">
    <button @click="open = !open">Toggle Reviews</button>
    <div x-show="open" style="display: none;">
        <h2>Reviews</h2>
        {% for review in object.review_set.all %}
        <div class="review">
            <h3>Review by {{ review.user.username }}</h3>
            <p>{{ review.content }}</p>
            <hr>
        </div>
        {% empty %}
        <p>No reviews yet.</p>
        {% endfor %}
    </div>
</div>
<a href="{% url 'create-review' %}?movie_id={{ object.pk }}">Write a Review</a>
{% endblock %}
```

#### `create_review.html`
For adding reviews.

```html
{% extends "base.html" %}
{% block title %}Add Review{% endblock %}
{% block content %}
<h1>Write a Review</h1>
<form method="post" hx-post="{% url 'create-review' %}" hx-target="#review-section">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Submit</button>
</form>
{% endblock %}
```

### 2. CSS Styling

Here is a basic CSS file for styling:

```css
/* styles.css */
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    line-height: 1.6;
}

.container {
    width: 80%;
    margin: 0 auto;
    padding: 20px;
}

nav {
    background: #333;
    color: #fff;
    padding: 10px 0;
}

nav ul {
    display: flex;
    justify-content: center;
    list-style: none;
    padding: 0;
}

nav ul li {
    margin: 0 15px;
}

nav ul li a {
    color: #fff;
    text-decoration: none;
}

.movie-card {
    background: #f4f4f4;
    margin: 10px 0;
    padding: 10px;
    border: 1px solid #ddd;
}

button {
    background-color: #333;
    color: white;
    padding: 10px 15px;
    border: none;
    cursor: pointer;
}

button:hover {
    background-color: #555;
}

footer {
    background: #333;
    color: #fff;
    text-align: center;
    padding: 10px 0;
    position: absolute;
    bottom: 0;
    width: 100%;
}
```

### 3. JavaScript and Interactivity

**HTMX:**

In the `movie_list.html`, the `<input>` will trigger an update to the movie list as the user types:

```html
<input type="text" id="search-bar" 
       hx-get="{% url 'movie-list' %}?q={query}"
       hx-trigger="keyup changed delay:500ms"
       hx-target="#movies-list"
       placeholder="Search Movies...">
```

**AlpineJS:**

In the `movie_detail.html`, use AlpineJS for interactivity in the review toggle feature:

```html
<div x-data="{ open: false }">
    <button @click="open = !open">Toggle Reviews</button>
    <div x-show="open" style="display: none;">
        <!-- Review details here -->
    </div>
</div>
```

### 4. Forms

For `create_review.html`, enhance form interactivity and submit via HTMX without a page reload:

```html
<form method="post" hx-post="{% url 'create-review' %}" hx-target="#review-section">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Submit</button>
</form>
```

### 5. Components

**Movie Card Component:** This is reusable across `movie_list.html`.

**Review Form Component:** A simple form structure to be reused for creating/editing reviews.

### 6. Navigation and User Flow

Ensure the navigation bar and links are functional and intuitive, facilitating seamless transitions between pages.

### 7. Error Handling and User Feedback

Display failure or success messages based on form submissions using HTMX.

### 8. Accessibility

- Utilize semantic tags appropriately (e.g., `<nav>`, `<main>`, `<article>`).
- Ensure all interactive elements are navigable using a keyboard.
- Use ARIA attributes where necessary to improve screen reader support.

### Final Integration

Ensure all templates are integrated into Django's template directory, link stylesheets and scripts properly, and test across different browsers to ensure compatibility and responsiveness. The combination of HTMX and AlpineJS facilitates an interactive experience without over-reliance on large JavaScript frameworks.
    
---
## QUIZ
## What is the purpose of 'context_object_name' in Django Class-Based Views?

In Django's Class-Based Views, `context_object_name` allows you to customize the name of the object passed to the template. Without it, the default name would be the model's lowercase name followed by `_list` or `_detail` depending on the view.  

In the tutorial, you see it used in `MovieListView`:

```python
context_object_name = 'movies'
```

This means that in `movie_list.html`, you can iterate over the queryset using `movies`.  If not specified, you would have to use `movie_list` instead.

## What is the purpose of the `{% static %}` template tag?

The `{% static %}` template tag helps you include static files like CSS, JavaScript, or images in your Django templates. It constructs the correct URL for serving these files, which is essential in production settings.

In `base.html`:

```html
<link rel="stylesheet" href="{% static 'css/styles.css' %}">
```

This line includes the `styles.css` file located inside the `static` folder of your app. Always use `{% static %}` to reference static files in your templates.

## What are the benefits of using HTMX's `hx-trigger` attribute?

The `hx-trigger` attribute in HTMX lets you control when an HTMX request should be triggered. This is useful for making your web app more responsive without writing a lot of JavaScript.

For example, in `movie_list.html`:

```html
hx-trigger="keyup changed delay:500ms"
```

This tells HTMX to send a request each time the user types something in the search bar, but only after they've stopped typing for 500 milliseconds. This avoids making unnecessary requests and improves performance.

## How does AlpineJS's `x-show` directive work?

The `x-show` directive in AlpineJS conditionally shows or hides elements on your page based on the truthiness of an expression. It provides a reactive way to manage element visibility.

Here's an example from `movie_detail.html`:

```html
<div x-show="open" style="display: none;">
```

This snippet will show the div containing the movie reviews only when the `open` variable is true.  The `style="display: none;"` ensures that the reviews are hidden by default.

## What does the `hx-target` attribute in HTMX do?

The `hx-target` attribute in HTMX specifies where to insert the response of an HTMX request on the page. This allows you to update specific parts of your page without a full reload.

For instance, in `movie_list.html`:

```html
hx-target="#movies-list"
```

This ensures that the server's response, which will be a new list of movies, will replace the content of the element with the ID `movies-list`. 

## What are the advantages of using a ManyToManyField in Django models?

A `ManyToManyField` in Django allows you to create a relationship where an object of one model can be associated with multiple objects of another model, and vice versa. 

In this tutorial, it's used to connect `Movie` and `Genre`:

```python
genres = models.ManyToManyField(Genre)
```

This means a movie can have multiple genres, and a genre can be associated with many movies. This is more flexible than a `ForeignKey`, which enforces a one-to-many relationship.
