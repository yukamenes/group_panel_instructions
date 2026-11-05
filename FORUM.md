# Group Panel — Completing the Forum Functionality

## Homework Guide

### What have we already done?

During the lesson, we started implementing a new feature called **Forum** in our Group Panel project.

We have already completed the following parts:

1. Created the `ForumTopic` model.
2. Created the `ForumReply` model.
3. Created the `forms.py` file.
4. Added two forms to `forms.py`:
   - `ForumTopicForm` — for creating new forum topics.
   - `ForumReplyForm` — for adding comments to existing topics.

**Important:** Do not create these models or forms again. They already exist in your project!

Your task is to complete the remaining functionality:

- Create views for the forum.
- Configure URL routes.
- Create HTML templates.
- Add a Forum link to the navigation menu.
- Test the entire functionality.

---

# Part 1. Understanding How the Forum Should Work

Before writing code, let's understand what we want to achieve.

Our forum will have three main pages:

| Page | Purpose |
|---|---|
| Forum page | Displays all existing topics. |
| Create topic page | Allows an authenticated user to create a new topic. |
| Topic details page | Displays one topic and all its comments. |

The functionality should work as follows:

1. A user opens the Forum page.
2. The user can see a list of topics.
3. The user clicks on a topic to open its details.
4. The user can read all comments under that topic.
5. An authenticated user can create a new topic.
6. An authenticated user can add a comment to an existing topic.

We will use the models and forms that we have already created.

## Understanding the relationship between models

Our models are connected:

- One `ForumTopic` can have many `ForumReply` objects.
- Each `ForumReply` belongs to one specific `ForumTopic`.
- Both topics and replies have an author.

This relationship allows us to display all comments related to a particular topic.

---

# Part 2. Creating Views

**File:** `core/views.py`

We will start by creating four views:

1. `forum` — displays all topics.
2. `create_topic` — creates a new topic.
3. `forum_topic_detail` — displays one topic and its replies.
4. `add_reply` — adds a reply to a topic.

Before writing code, open `core/views.py`.

Find the existing imports at the top of the file.

## Step 1. Add the necessary imports

Add the following imports if they are not already present:

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required

from .models import ForumTopic, ForumReply
from .forms import ForumTopicForm, ForumReplyForm
```

### Explanation

Let's understand what each import does.

**`render`**

This function displays an HTML template and passes data from a view to that template.

For example:

```python
return render(request, "core/forum.html", {"topics": topics})
```

Here, Django opens `forum.html` and makes the `topics` variable available inside the template.

**`redirect`**

This function redirects the user to another URL.

For example:

```python
return redirect("forum")
```

After successfully creating a topic, we can redirect the user back to the forum page.

**`get_object_or_404`**

This function searches for an object in the database.

If the object exists, it returns it.

If the object does not exist, Django automatically returns a 404 error page.

**`login_required`**

This decorator protects a view so that only authenticated users can access it.

If someone is not logged in, Django redirects them to the login page.

**Models and forms**

We import our existing models and forms so that we can use them inside our views.

---

## Step 2. Create a view for displaying all forum topics

Add the following function to `core/views.py`:

```python
def forum(request):
    topics = ForumTopic.objects.select_related("author").order_by("-created_at")

    return render(request, "core/forum.html", {
        "topics": topics
    })
```

### Explanation

Let's examine the code.

**1. Creating the `forum` function**

```python
def forum(request):
```

This function will handle requests to the main Forum page.

The `request` parameter contains information about the current HTTP request.

**2. Getting topics from the database**

```python
topics = ForumTopic.objects.select_related("author").order_by("-created_at")
```

Let's break this line into smaller parts.

`ForumTopic.objects` gives us access to the database records of the `ForumTopic` model.

`select_related("author")` tells Django to retrieve the related author information together with each topic. This helps avoid unnecessary additional database queries when displaying authors.

`order_by("-created_at")` sorts topics by their creation date.

The minus sign means descending order, so the newest topics appear first.

**3. Displaying the HTML page**

```python
return render(request, "core/forum.html", {
    "topics": topics
})
```

We display the `forum.html` template and pass the list of topics to it.

The dictionary contains the data that will be available inside the HTML template.

### Checkpoint 1 — Test the view

Before continuing, save your file.

At this stage, the HTML template does not exist yet, so opening the Forum page will not work.

For now, check that:

- There are no syntax errors in `views.py`.
- The model and form imports are correct.
- The function is outside any other function.

We will test this view properly after creating the HTML template.

---

## Step 3. Create a view for adding new topics

Below the previous function, add:

```python
@login_required
def create_topic(request):
    if request.method == "POST":
        form = ForumTopicForm(request.POST)

        if form.is_valid():
            topic = form.save(commit=False)
            topic.author = request.user
            topic.save()

            return redirect("forum")
    else:
        form = ForumTopicForm()

    return render(request, "core/create_topic.html", {
        "form": form
    })
```

### Explanation

This view allows an authenticated user to create a new forum topic.

**1. Protecting the view**

```python
@login_required
```

This decorator means that only logged-in users can create topics.

Unauthenticated users will be redirected to the login page.

**2. Checking the request method**

```python
if request.method == "POST":
```

When a user submits a form, the browser sends a POST request.

We check whether the current request is POST because we only want to save data after the user submits the form.

**3. Creating a form with submitted data**

```python
form = ForumTopicForm(request.POST)
```

`request.POST` contains the information entered by the user.

We pass this information into our existing `ForumTopicForm`.

**4. Validating the form**

```python
if form.is_valid():
```

Django checks whether the submitted information satisfies the validation rules defined in the form and model.

If the data is valid, we can save it.

**5. Preparing the topic object**

```python
topic = form.save(commit=False)
```

Normally, `form.save()` saves the object directly to the database.

However, `commit=False` tells Django:

"Create the object, but do not save it to the database yet."

Why do we need this?

Because our model also requires an author, and the author is not entered manually in the form.

**6. Assigning the current user**

```python
topic.author = request.user
```

`request.user` represents the currently logged-in user.

We assign this user as the author of the new topic.

**7. Saving the topic**

```python
topic.save()
```

Now the object contains all required information, so we can save it to the database.

**8. Redirecting the user**

```python
return redirect("forum")
```

After successfully creating a topic, the user returns to the main Forum page.

**9. Displaying an empty form**

```python
else:
    form = ForumTopicForm()
```

If the request is not POST, it means that the user has opened the page for the first time.

We create an empty form for the user to fill in.

Finally:

```python
return render(request, "core/create_topic.html", {
    "form": form
})
```

We display the HTML page and pass the form to it.

### Checkpoint 2 — Understand the logic

Answer these questions before continuing:

1. Why do we use `@login_required`?
2. What is the difference between GET and POST requests in this view?
3. Why do we use `commit=False`?
4. What does `request.user` contain?
5. Why do we redirect the user after successfully saving a topic?

---

## Step 4. Create a view for displaying topic details

Add the following function:

```python
def forum_topic_detail(request, topic_id):
    topic = get_object_or_404(
        ForumTopic.objects.select_related("author"),
        id=topic_id
    )

    replies = topic.replies.select_related("author").order_by("created_at")

    return render(request, "core/forum_topic_detail.html", {
        "topic": topic,
        "replies": replies
    })
```

### Explanation

This view displays one selected topic and all comments related to it.

**1. Function parameters**

```python
def forum_topic_detail(request, topic_id):
```

We receive two parameters:

- `request` — information about the current request.
- `topic_id` — the ID of the topic we want to display.

For example, if the user opens `/forum/5/`, Django will pass `5` as `topic_id`.

**2. Finding the topic**

```python
topic = get_object_or_404(
    ForumTopic.objects.select_related("author"),
    id=topic_id
)
```

We search for a topic with the specified ID.

If the topic does not exist, Django returns a 404 page instead of causing an error.

**3. Getting comments**

```python
replies = topic.replies.select_related("author").order_by("created_at")
```

Remember that our `ForumReply` model has a relationship with `ForumTopic`.

We previously used `related_name="replies"` in the model.

Because of this, we can access all replies belonging to a topic using:

```python
topic.replies.all()
```

In our code, we also use:

- `select_related("author")` to retrieve author information efficiently.
- `order_by("created_at")` to display comments from oldest to newest.

**4. Passing information to the template**

```python
return render(request, "core/forum_topic_detail.html", {
    "topic": topic,
    "replies": replies
})
```

We pass two objects:

- `topic` — the selected topic.
- `replies` — all comments belonging to that topic.

The HTML template will use this information to display the page.

---

## Step 5. Create a view for adding comments

Add the final view:

```python
@login_required
def add_reply(request, topic_id):
    topic = get_object_or_404(ForumTopic, id=topic_id)

    if request.method == "POST":
        form = ForumReplyForm(request.POST)

        if form.is_valid():
            reply = form.save(commit=False)
            reply.topic = topic
            reply.author = request.user
            reply.save()

    return redirect("forum_topic_detail", topic_id=topic.id)
```

### Explanation

This view allows an authenticated user to add a comment to a specific topic.

**1. Protecting the view**

```python
@login_required
```

Only authenticated users can add comments.

**2. Finding the topic**

```python
topic = get_object_or_404(ForumTopic, id=topic_id)
```

We find the topic to which the user wants to add a comment.

We need this object because every reply must belong to a specific topic.

**3. Checking the request method**

```python
if request.method == "POST":
```

We only process submitted data when the user sends a POST request.

**4. Creating and validating the form**

```python
form = ForumReplyForm(request.POST)

if form.is_valid():
```

We create a form using the submitted information and check whether it is valid.

**5. Preparing the reply**

```python
reply = form.save(commit=False)
```

We create the reply object without saving it immediately.

We still need to assign its topic and author.

**6. Connecting the reply with a topic**

```python
reply.topic = topic
```

This line connects the new reply to the topic that the user opened.

**7. Assigning the author**

```python
reply.author = request.user
```

The currently logged-in user becomes the author of the reply.

**8. Saving the reply**

```python
reply.save()
```

Now we can save the comment to the database.

**9. Returning to the topic page**

```python
return redirect("forum_topic_detail", topic_id=topic.id)
```

After submitting a comment, the user returns to the same topic page.

This allows them to see the comment they have just added.

### Checkpoint 3 — Test your understanding

Explain:

1. Why do we need `topic_id` in this view?
2. Why do we assign `reply.topic = topic`?
3. Why do we assign `reply.author = request.user`?
4. What happens after the reply is saved?

**Important:** Make sure all four views are written at the top level of `views.py`, not inside another function.

---

# Part 3. Configuring URL Routes

**File:** `core/urls.py`

Now we need to connect our views with URLs.

Open `core/urls.py`.

If this file does not exist, create it inside the `core` application folder.

Add the following code:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("forum/", views.forum, name="forum"),
    path("forum/create/", views.create_topic, name="create_topic"),
    path("forum/<int:topic_id>/", views.forum_topic_detail, name="forum_topic_detail"),
    path("forum/<int:topic_id>/reply/", views.add_reply, name="add_reply"),
]
```

### Explanation

Let's understand the routes one by one.

**1. Main Forum page**

```python
path("forum/", views.forum, name="forum"),
```

When a user opens `/forum/`, Django calls the `forum` view.

**2. Creating a topic**

```python
path("forum/create/", views.create_topic, name="create_topic"),
```

This URL opens the page where users can create new topics.

**3. Topic details**

```python
path("forum/<int:topic_id>/", views.forum_topic_detail, name="forum_topic_detail"),
```

The `<int:topic_id>` part is a dynamic URL parameter.

For example:

- `/forum/1/` — displays topic with ID 1.
- `/forum/2/` — displays topic with ID 2.
- `/forum/15/` — displays topic with ID 15.

Django extracts the number and passes it to the view as `topic_id`.

**4. Adding a reply**

```python
path("forum/<int:topic_id>/reply/", views.add_reply, name="add_reply"),
```

This URL handles the submission of a new comment for a particular topic.

### Important: Connect application URLs to the project

Open the main project URL file:

`group_portal/urls.py`

Find the existing `urlpatterns` list.

If your application URLs are not included yet, add:

```python
from django.urls import path, include
```

Then add this route inside `urlpatterns`:

```python
path("", include("core.urls")),
```

For example:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("core.urls")),
]
```

**Important:** If your project already uses another URL configuration, do not replace it. Keep the existing routes and add the Forum routes without creating duplicates.

If your project currently defines application routes directly in `group_portal/urls.py`, you can add the four Forum routes there instead. In that case, do not create a second URL configuration for the same routes.

### Checkpoint 4 — Test the URLs

Run the development server:

```bash
python manage.py runserver
```

Check that Django does not report any URL configuration errors.

The pages may not display correctly yet because we have not created the templates.

---

# Part 4. Creating HTML Templates

Now we need to create three HTML files.

All three templates should be located in:

`core/templates/core/`

Your folder structure should look like this:

```text
core/
│
├── templates/
│   └── core/
│       ├── forum.html
│       ├── create_topic.html
│       └── forum_topic_detail.html
│
├── models.py
├── forms.py
├── views.py
└── urls.py
```

If the `templates` or inner `core` folder does not exist, create it.

We will use Django Template Language to display information from our views.

## Step 1. Create `forum.html`

**File:** `core/templates/core/forum.html`

Create a new HTML file called `forum.html`.

Add:

```html
{% extends "core/base.html" %}

{% block content %}

<h1>Forum</h1>

<a href="{% url 'create_topic' %}">Create a new topic</a>

<hr>

{% for topic in topics %}

    <div>
        <h2>
            <a href="{% url 'forum_topic_detail' topic.id %}">
                {{ topic.title }}
            </a>
        </h2>

        <p>{{ topic.content }}</p>

        <p>
            Author: {{ topic.author.username }}
        </p>

        <p>
            Created: {{ topic.created_at }}
        </p>

        <p>
            Comments: {{ topic.replies.count }}
        </p>
    </div>

    <hr>

{% empty %}

    <p>There are no topics yet.</p>

{% endfor %}

{% endblock %}
```

### Explanation

**1. Extending the base template**

```html
{% extends "core/base.html" %}
```

We use our existing `base.html` template.

This means that the Forum page will inherit the common layout of our project, including its navigation menu and general structure.

**2. Defining the content block**

```html
{% block content %}
```

We place the page content inside the block provided by `base.html`.

At the end of the file, we close it:

```html
{% endblock %}
```

**3. Creating a link to the topic creation page**

```html
<a href="{% url 'create_topic' %}">Create a new topic</a>
```

The `{% url %}` template tag generates a URL using the route name.

We use the name `create_topic` that we defined in `urls.py`.

**4. Displaying all topics**

```html
{% for topic in topics %}
```

This is a Django template loop.

It goes through every topic passed from our `forum` view.

Inside the loop, we can access the properties of each topic:

```html
{{ topic.title }}
{{ topic.content }}
{{ topic.author.username }}
```

Double curly brackets `{{ }}` are used to display variable values.

**5. Creating a link to topic details**

```html
<a href="{% url 'forum_topic_detail' topic.id %}">
    {{ topic.title }}
</a>
```

When the user clicks on the topic title, they will open its details page.

We pass `topic.id` because our URL requires a topic ID.

**6. Displaying the number of comments**

```html
{{ topic.replies.count }}
```

We use the `replies` relationship defined in our model.

Django counts how many replies belong to the current topic.

**7. Handling an empty list**

```html
{% empty %}
    <p>There are no topics yet.</p>
```

The `{% empty %}` block is displayed when the loop has no objects to iterate over.

This is useful when our database does not contain any topics yet.

### Test 1

1. Start the development server.
2. Open `http://127.0.0.1:8000/forum/`.
3. Check whether the Forum page opens.
4. If you already have topics in your database, check whether they are displayed.
5. If there are no topics, check whether the message "There are no topics yet" appears.

If you receive a `TemplateDoesNotExist` error, check the file name and folder structure.

---

## Step 2. Create `create_topic.html`

**File:** `core/templates/core/create_topic.html`

Create another HTML file.

Add:

```html
{% extends "core/base.html" %}

{% block content %}

<h1>Create a new topic</h1>

<form method="POST">

    {% csrf_token %}

    {{ form.as_p }}

    <button type="submit">Create topic</button>

</form>

<a href="{% url 'forum' %}">Back to Forum</a>

{% endblock %}
```

### Explanation

**1. Extending the base template**

```html
{% extends "core/base.html" %}
```

We use the same layout as the rest of the project.

**2. Creating a form**

```html
<form method="POST">
```

We create an HTML form that sends information using the POST method.

This matches the logic in our `create_topic` view.

**3. Adding CSRF protection**

```html
{% csrf_token %}
```

Django requires a CSRF token for POST forms.

This token protects our application against certain types of unauthorized requests.

Always include it inside forms that send POST requests.

**4. Displaying Django form fields**

```html
{{ form.as_p }}
```

This is a convenient Django method that displays all fields of the form.

The fields are wrapped in paragraph tags.

Because we already created `ForumTopicForm`, Django knows which fields should be displayed.

**5. Creating the submit button**

```html
<button type="submit">Create topic</button>
```

When the user clicks this button, the browser submits the form.

**6. Returning to the Forum page**

```html
<a href="{% url 'forum' %}">Back to Forum</a>
```

This link allows the user to return to the main Forum page.

### Test 2

1. Open `http://127.0.0.1:8000/forum/`.
2. Click **Create a new topic**.
3. Enter a title and some content.
4. Click **Create topic**.
5. You should be redirected to the Forum page.
6. Check whether your new topic appears in the list.
7. Open the topic and verify its information.
8. Open the Django admin panel or inspect the database to confirm that the topic was saved.

If the form displays validation errors, check whether all required fields have been completed.

---

## Step 3. Create `forum_topic_detail.html`

**File:** `core/templates/core/forum_topic_detail.html`

This page will display one topic and all comments associated with it.

Create the file and add:

```html
{% extends "core/base.html" %}

{% block content %}

<h1>{{ topic.title }}</h1>

<p>{{ topic.content }}</p>

<p>
    Author: {{ topic.author.username }}
</p>

<p>
    Created: {{ topic.created_at }}
</p>

<hr>

<h2>Comments</h2>

{% for reply in replies %}

    <div>
        <p>{{ reply.content }}</p>

        <p>
            Author: {{ reply.author.username }}
        </p>

        <p>
            Created: {{ reply.created_at }}
        </p>
    </div>

    <hr>

{% empty %}

    <p>There are no comments yet.</p>

{% endfor %}

{% if user.is_authenticated %}

    <h2>Add a comment</h2>

    <form method="POST" action="{% url 'add_reply' topic.id %}">

        {% csrf_token %}

        {{ reply_form.as_p }}

        <button type="submit">Add comment</button>

    </form>

{% else %}

    <p>
        Please log in to add a comment.
    </p>

{% endif %}

<a href="{% url 'forum' %}">Back to Forum</a>

{% endblock %}
```

### Explanation

Let's understand the most important parts.

**1. Displaying topic information**

```html
<h1>{{ topic.title }}</h1>

<p>{{ topic.content }}</p>
```

We display the title and content of the selected topic.

The `topic` variable comes from our `forum_topic_detail` view.

**2. Displaying all comments**

```html
{% for reply in replies %}
```

We use a loop to display every comment passed from the view.

Inside the loop:

```html
{{ reply.content }}
{{ reply.author.username }}
{{ reply.created_at }}
```

These expressions display the comment text, its author and creation date.

**3. Handling topics without comments**

```html
{% empty %}
    <p>There are no comments yet.</p>
```

If there are no replies, the user will see a message instead of an empty section.

**4. Checking whether the user is authenticated**

```html
{% if user.is_authenticated %}
```

This condition checks whether the current user is logged in.

If the user is authenticated, we display the comment form.

Otherwise, we display a message asking them to log in.

**5. Creating the comment form**

```html
<form method="POST" action="{% url 'add_reply' topic.id %}">
```

This form sends a POST request to the `add_reply` URL.

Notice that we pass `topic.id`.

This is important because the comment must be connected to the correct topic.

**6. Displaying the form**

```html
{{ reply_form.as_p }}
```

We display the `ForumReplyForm` passed from the view.

**Important:** Our current `forum_topic_detail` view must also pass this form to the template.

We will update it in the next step.

---

## Step 4. Update the topic details view

**File:** `core/views.py`

Find the `forum_topic_detail` function that we created earlier.

Replace its current code with:

```python
def forum_topic_detail(request, topic_id):
    topic = get_object_or_404(
        ForumTopic.objects.select_related("author"),
        id=topic_id
    )

    replies = topic.replies.select_related("author").order_by("created_at")

    reply_form = ForumReplyForm()

    return render(request, "core/forum_topic_detail.html", {
        "topic": topic,
        "replies": replies,
        "reply_form": reply_form
    })
```

### Why do we need this change?

Previously, our view passed only two variables:

- `topic`
- `replies`

However, our HTML template now also needs `reply_form`.

This line creates an empty comment form:

```python
reply_form = ForumReplyForm()
```

Then we add it to the context dictionary:

```python
"reply_form": reply_form
```

Now the HTML template can access it using:

```html
{{ reply_form.as_p }}
```

**Remember:** Every variable used in a template must be available in the context passed by its view, unless it is provided by a context processor.

### Test 3

1. Open the Forum page.
2. Click on one of your topics.
3. Check whether its title and content are displayed.
4. Check whether existing comments appear.
5. If you are logged in, check whether the comment form is visible.
6. If you are logged out, check whether the login message appears.

---

# Part 5. Adding Forum to the Navigation Menu

**File:** `core/templates/core/base.html`

Open the existing `base.html` file.

Find the navigation menu where you already have links such as:

- Home
- Announcements
- Events

Add another link:

```html
<a href="{% url 'forum' %}">Forum</a>
```

For example, your navigation menu might look similar to this:

```html
<nav>
    <a href="{% url 'home' %}">Home</a>

    <a href="{% url 'announcement_list' %}">
        Announcements
    </a>

    <a href="{% url 'event_list' %}">
        Events
    </a>

    <a href="{% url 'forum' %}">
        Forum
    </a>
</nav>
```

**Important:** The names of your existing routes may be different. Keep the names already used in your project and add only the Forum link.

### Test 4

1. Open the home page.
2. Find the navigation menu.
3. Click the Forum link.
4. Check whether the Forum page opens correctly.

---

# Part 6. Complete Forum Testing

Now we need to test the entire functionality, not just individual pages.

Run the development server:

```bash
python manage.py runserver
```

Open the application in your browser.

## Test 1. Viewing the Forum

- Open the Forum page.
- Check whether all existing topics are displayed.
- Check whether the newest topics appear first.
- If there are no topics, check whether the empty-state message appears.

## Test 2. Creating a topic

- Log in to your account.
- Open the Forum page.
- Click **Create a new topic**.
- Enter a title.
- Enter the topic content.
- Submit the form.
- Check whether you return to the Forum page.
- Check whether your new topic appears in the list.
- Open the topic and verify its information.

## Test 3. Adding comments

- Open an existing topic.
- Find the comment form.
- Enter a comment.
- Click **Add comment**.
- Check whether the page redirects to the same topic.
- Check whether your new comment appears.
- Verify that the correct author is displayed.

## Test 4. Authentication

We also need to check what happens when a user is not logged in.

1. Log out of your account.
2. Open the Forum page.
3. Check whether you can still read topics.
4. Try to open `/forum/create/`.
5. Check whether Django redirects you to the login page.
6. Open a topic.
7. Check whether the comment form is hidden.
8. Try to open the reply URL directly in the browser using a topic ID, for example `/forum/1/reply/`.

The last request should not create a comment because adding replies requires an authenticated user and a POST request.

---

# Part 7. Troubleshooting

Here are some common problems you may encounter.

| Error | Possible reason | What to check |
|---|---|---|
| `ModuleNotFoundError` | Incorrect import | Check model and form imports. |
| `TemplateDoesNotExist` | Incorrect template location | Check the `templates/core/` folder. |
| `NoReverseMatch` | Incorrect URL name | Compare `{% url %}` names with `urls.py`. |
| `NameError` | Variable or function is not defined | Check spelling and imports. |
| `OperationalError` | Database or table problem | Check whether migrations have been applied. |
| `403 Forbidden` | CSRF token is missing | Check whether `{% csrf_token %}` is inside the form. |
| `404 Not Found` | The requested topic does not exist | Check the topic ID and URL pattern. |

### Useful commands

If Django reports that a database table is missing, check your migrations:

```bash
python manage.py showmigrations
```

If migrations are required, run:

```bash
python manage.py makemigrations
python manage.py migrate
```

Since our models were already created during the previous work, do not create duplicate models. Only make migrations if Django detects actual model changes that have not been migrated.

---

# Part 8. Final Checklist

Before submitting your homework, make sure everything works.

- [ ] I did not recreate `ForumTopic` or `ForumReply`.
- [ ] I used the existing forms from `forms.py`.
- [ ] I created all four views.
- [ ] I configured all four URL routes.
- [ ] I connected the application URLs to the project.
- [ ] I created `forum.html`.
- [ ] I created `create_topic.html`.
- [ ] I created `forum_topic_detail.html`.
- [ ] I updated the topic details view to pass `reply_form`.
- [ ] I added the Forum link to `base.html`.
- [ ] I tested displaying topics.
- [ ] I tested creating a topic.
- [ ] I tested adding comments.
- [ ] I tested the functionality without logging in.
- [ ] I checked that there are no errors in the terminal.

---

# Important: How to Work on This Homework

The goal of this assignment is not simply to copy code into your project.

For every step:

1. Read the explanation before writing the code.
2. Try to understand what the code does and why we need it.
3. Type the code yourself instead of copying the entire solution.
4. Save the file and check whether there are any errors.
5. Run the project and test the functionality in your browser.
6. If something does not work, read the error message carefully and try to identify its cause.
7. If you cannot solve the problem, write down the error and the steps you have already tried.

**Be prepared to explain during our next lesson:**

- What is a Django view?
- What is the difference between GET and POST requests?
- Why do we use `login_required`?
- What does `get_object_or_404()` do?
- Why do we use `commit=False` when creating a topic or reply?
- How are `ForumTopic` and `ForumReply` connected?
- How does Django know which HTML template to display?
- How does a URL route connect a browser request to a view?

The most important thing is to understand the code you write. If you can explain how your functionality works, it means you are making real progress as a programmer!

Good luck! 😊
