# Group Panel — Homework: Complete the Polls Module

Hi, Anna! 😊

During the lesson, we started implementing the **Polls** feature in our Group Panel project.

You have already completed these steps:

- Created the `Poll` model.
- Created the `PollOption` model.
- Created and applied database migrations.
- Registered both models in `core/admin.py`.

Your task is to finish the Polls module so that moderators can create polls and users can view them on the website.

> **Important:** Do not just copy the code. Read each explanation, understand what every part does, and try to implement the steps yourself. At the end of this guide, there are self-check questions. Be ready to explain your code at the beginning of our next lesson.

---

## 1. What should the finished feature do?

When you finish the homework:

1. The website should have a **Polls** link in its navigation.
2. Users should be able to open the Polls page and see all available polls.
3. Each poll should display its question and answer options.
4. Moderators and superusers should see a **Create a poll** link.
5. Moderators and superusers should be able to create a poll with three answer options.
6. Regular users should not be able to create polls, even if they manually enter the creation page URL.
7. Unauthenticated users should be asked to log in before opening the poll creation page.

For this homework, users only need to **view** polls. Voting and poll results will be implemented later.

---

## 2. Check your existing models

Open:

`core/models.py`

You should already have the following models from the lesson:

```python
class Poll(models.Model):
    question = models.CharField(max_length=255)

    created_by = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name="created_polls"
    )

    created_at = models.DateTimeField(auto_now_add=True)
    is_active = models.BooleanField(default=True)

    def __str__(self):
        return self.question


class PollOption(models.Model):
    poll = models.ForeignKey(
        Poll,
        on_delete=models.CASCADE,
        related_name="options"
    )

    text = models.CharField(max_length=255)

    def __str__(self):
        return self.text
```

You do **not** need to create these models again.

### Understand the relationship

One poll can have several answer options. Each answer option belongs to one poll.

For example:

```python
poll = Poll.objects.get(id=1)

options = poll.options.all()
```

`poll.options.all()` retrieves all answer options connected to that poll. This works because we defined `related_name="options"` in the `PollOption` model.

---

## 3. Create the PollForm

Open:

`core/forms.py`

Find the imports at the top of the file. Make sure `Poll` and `PollOption` are imported. Do not add duplicate imports if they are already there.

```python
from .models import Poll, PollOption
```

Now create a new `PollForm` below your existing forms:

```python
class PollForm(forms.ModelForm):
    option_one = forms.CharField(
        max_length=255,
        label="First option"
    )

    option_two = forms.CharField(
        max_length=255,
        label="Second option"
    )

    option_three = forms.CharField(
        max_length=255,
        label="Third option"
    )

    class Meta:
        model = Poll
        fields = ["question"]
```

### Explanation

`PollForm` is a `ModelForm`, because the main question is saved in the `Poll` model.

The three `option_...` fields are additional form fields. They are not fields of the `Poll` model; they collect data that we will use to create separate `PollOption` objects.

The `Meta` class tells Django which model the form is connected to and which model fields should be included automatically.

### Checkpoint

Before continuing, make sure you can explain:

- Why do we use `ModelForm`?
- Why are the three answer fields not included in `Meta.fields`?
- What is the difference between a model field and an additional form field?

---

## 4. Create the views

Open:

`core/views.py`

### 4.1. Check the imports

Make sure these imports are present. If your file already imports these names, do not duplicate them.

```python
from .models import Poll, PollOption
from .forms import PollForm
```

You will also need the functions and decorators used by the rest of your views. Your project may already import them:

```python
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
```

The project already has a permission helper called `can_manage_announcements(user)`. It returns `True` for superusers and members of the `Moderators` group. We will reuse it for polls.

### 4.2. Create a view that displays polls

Add this view:

```python
def polls(request):
    all_polls = Poll.objects.all().order_by("-created_at")

    can_create = (
        request.user.is_authenticated
        and can_manage_announcements(request.user)
    )

    return render(request, "core/polls.html", {
        "polls": all_polls,
        "can_create": can_create
    })
```

#### How does it work?

- `Poll.objects.all()` retrieves all poll objects from the database.
- `.order_by("-created_at")` sorts polls by creation date, newest first. The minus sign means descending order.
- `can_create` checks whether the current user is authenticated and has permission to manage polls.
- `render()` opens the HTML template and passes the polls and permission value to it.

We pass `can_create` to the template so that the template can decide whether to display the creation link.

### 4.3. Create a view for creating polls

Below the previous view, add:

```python
@login_required
def create_poll(request):
    if not can_manage_announcements(request.user):
        return redirect("polls")

    if request.method == "POST":
        form = PollForm(request.POST)

        if form.is_valid():
            poll = form.save(commit=False)
            poll.created_by = request.user
            poll.save()

            PollOption.objects.create(
                poll=poll,
                text=form.cleaned_data["option_one"]
            )

            PollOption.objects.create(
                poll=poll,
                text=form.cleaned_data["option_two"]
            )

            PollOption.objects.create(
                poll=poll,
                text=form.cleaned_data["option_three"]
            )

            return redirect("polls")

    else:
        form = PollForm()

    return render(request, "core/create_poll.html", {
        "form": form
    })
```

#### How does it work?

**1. Authentication**

```python
@login_required
```

This decorator requires the user to log in before accessing the view.

**2. Permission check**

```python
if not can_manage_announcements(request.user):
    return redirect("polls")
```

Only moderators and superusers can create polls.

It is important to check permissions in the view. Hiding a link in HTML is not enough: a user could still try to open the URL manually.

**3. Check the request method**

```python
if request.method == "POST":
    form = PollForm(request.POST)
```

A POST request means that the user has submitted the form. We pass the submitted data to the form.

If the user opens the page for the first time, the request is GET, so we create an empty form:

```python
else:
    form = PollForm()
```

**4. Validate the form**

```python
if form.is_valid():
```

Django checks whether the submitted values are valid and whether required fields have been filled in.

**5. Save the poll**

```python
poll = form.save(commit=False)
poll.created_by = request.user
poll.save()
```

- `commit=False` creates the model object without saving it immediately.
- We assign the currently logged-in user as the poll creator.
- `poll.save()` saves the poll to the database.

**6. Save the answer options**

For example:

```python
PollOption.objects.create(
    poll=poll,
    text=form.cleaned_data["option_one"]
)
```

This creates a new `PollOption` object.

- `poll=poll` connects the option to the poll we have just created.
- `text=...` takes the validated answer text from the form.

The next two blocks do the same thing for the second and third options.

**What is `cleaned_data`?**

After `form.is_valid()` returns `True`, Django makes the validated form values available through `form.cleaned_data`.

For example:

```python
form.cleaned_data["option_one"]
```

returns the value entered in the first answer field.

### Checkpoint

Explain these questions in your own words:

1. Why do we use `@login_required`?
2. Why do we check `can_manage_announcements()` in the view?
3. What is the difference between GET and POST?
4. Why do we use `form.is_valid()`?
5. Why do we save the poll before creating its options?
6. What does `form.cleaned_data` contain?

---

## 5. Add URL patterns

Find where the existing project URLs are configured.

If your project has a separate `core/urls.py`, add the routes there. If your existing routes are defined directly in `group_portal/urls.py`, add them to that file instead.

**Do not create duplicate URL configurations.** Follow the structure already used by your project.

Add these two paths to the existing `urlpatterns` list:

```python
path("polls/", views.polls, name="polls"),
path("polls/create/", views.create_poll, name="create_poll"),
```

Make sure `path` and `views` are imported in the same way as your existing routes.

### What do these paths mean?

- `polls/` opens the page with the list of polls.
- `polls/create/` opens the page with the poll creation form.
- `name="polls"` and `name="create_poll"` are URL names. We can use them in templates with Django's `{% url %}` tag.

---

## 6. Create the HTML templates

Create both files inside:

`core/templates/core/`

Your folder structure should look similar to this:

```text
core/
└── templates/
    └── core/
        ├── base.html
        ├── polls.html
        └── create_poll.html
```

Do not replace your existing `base.html`. We will extend it.

### 6.1. Create `polls.html`

Create a file named:

`core/templates/core/polls.html`

Add:

```html
{% extends "core/base.html" %}

{% block content %}

<h1>Polls</h1>

{% if can_create %}
    <a href="{% url 'create_poll' %}">Create a poll</a>
{% endif %}

{% for poll in polls %}

    <div class="poll">
        <h2>{{ poll.question }}</h2>

        <p>Created by: {{ poll.created_by }}</p>
        <p>Date: {{ poll.created_at }}</p>

        <ul>
            {% for option in poll.options.all %}
                <li>{{ option.text }}</li>
            {% endfor %}
        </ul>
    </div>

{% empty %}

    <p>No polls available yet.</p>

{% endfor %}

{% endblock %}
```

#### Explanation

- `{% extends "core/base.html" %}` tells Django that this page uses the existing base template.
- `{% block content %}` defines the content that will be inserted into the content block of `base.html`.
- `{% if can_create %}` displays the creation link only if the view passed `True`.
- `{% for poll in polls %}` loops through all polls passed by the view.
- `{{ poll.question }}` displays the question.
- `{% for option in poll.options.all %}` loops through the options belonging to the current poll.
- `{{ option.text }}` displays the answer text.
- `{% empty %}` displays a message if there are no polls.

**Important:** Do not replace the `can_create` condition with a check such as `user.groups.all`. We already performed the correct permission check in the view and passed its result to the template.

### 6.2. Create `create_poll.html`

Create a file named:

`core/templates/core/create_poll.html`

Add:

```html
{% extends "core/base.html" %}

{% block content %}

<h1>Create a Poll</h1>

<form method="POST">

    {% csrf_token %}

    {{ form.as_p }}

    <button type="submit">Create Poll</button>

</form>

{% endblock %}
```

#### Explanation

- `<form method="POST">` sends the form data to the server using a POST request.
- `{% csrf_token %}` adds Django's CSRF protection token. Django requires it for POST forms when CSRF protection is enabled.
- `{{ form.as_p }}` displays the fields from the Django form, each wrapped in a paragraph.
- The submit button sends the form.

---

## 7. Add Polls to the navigation

Open:

`core/templates/core/base.html`

Find the section where the navigation links are displayed.

Add the following link next to the existing links for Home, Announcements, Events, and Forum:

```html
<a href="{% url 'polls' %}">Polls</a>
```

Do not remove or rewrite the existing navigation. Just add the new link in the appropriate place.

---

## 8. Test your work

Run the development server:

```bash
python manage.py runserver
```

Open the website in your browser and test each scenario.

### Test 1 — View the Polls page

1. Click the **Polls** link in the navigation.
2. Check that the page opens without errors.
3. Check that the poll created in the Django admin is displayed.
4. Check that its question and all answer options are visible.

If there are no polls, the page should display:

`No polls available yet.`

### Test 2 — Create a poll as a moderator

1. Log in as a user who belongs to the `Moderators` group.
2. Open the Polls page.
3. Check that the **Create a poll** link is visible.
4. Open the creation form.
5. Enter a question and three answer options.
6. Submit the form.
7. Check that you are redirected to the Polls page.
8. Check that the new poll and all three options are displayed.

### Test 3 — Check regular user permissions

1. Log in as a regular user who is not a moderator or superuser.
2. Open the Polls page.
3. Check that the **Create a poll** link is not visible.
4. Try to open `/polls/create/` directly in the browser.
5. Check that you are redirected to the Polls page and cannot create a poll.

### Test 4 — Check unauthenticated access

1. Log out.
2. Try to open `/polls/create/`.
3. Check that Django redirects you to the login page.

### Test 5 — Check the database

Open the Django admin panel.

Check that:
- Your new poll appears in the `Poll` model.
- Its three answer options appear in `PollOption`.
- Each option is connected to the correct poll.

---

## 9. Troubleshooting

If something does not work, do not immediately copy the code again. Read the error message and try to identify where the problem occurs.

### `NoReverseMatch`

Possible reason: the URL name used in the template does not match the name in `urls.py`.

Check that you have:

```python
name="polls"
```

and:

```python
name="create_poll"
```

### `TemplateDoesNotExist`

Possible reason: the template is in the wrong folder or its name is different.

Check the paths:

```text
core/templates/core/polls.html
core/templates/core/create_poll.html
```

### `NameError: name 'Poll' is not defined`

Possible reason: `Poll` was not imported in `views.py` or `forms.py`.

Check the imports at the top of the file where the error occurs.

### `ImportError` or duplicate imports

Check that you have not accidentally created duplicate or conflicting imports. Keep the existing project structure.

### The Create a poll link is visible to the wrong users

Check that:
- The `polls` view calculates `can_create` using `can_manage_announcements(request.user)`.
- The template uses `{% if can_create %}`.
- The permission helper checks for superusers and the `Moderators` group.

### The poll is created, but options are missing

Check that:
- The poll is saved before its options are created.
- Each `PollOption.objects.create()` call receives `poll=poll`.
- The form field names match the names used in `cleaned_data`.

---

## 10. Self-check questions

Prepare answers to these questions. You should be able to explain them without simply reading the code.

1. What is the purpose of the `Poll` model?
2. What is the purpose of the `PollOption` model?
3. What type of relationship exists between `Poll` and `PollOption`?
4. What does `related_name="options"` allow us to do?
5. Why do we use a `ModelForm` for `PollForm`?
6. Why are `option_one`, `option_two`, and `option_three` additional form fields?
7. What is the difference between GET and POST requests?
8. What does `form.is_valid()` check?
9. What is `form.cleaned_data`?
10. Why do we use `form.save(commit=False)`?
11. Why do we assign `poll.created_by = request.user`?
12. Why must the poll be saved before its options?
13. Why do we check permissions in the view?
14. Why is hiding the creation link in the template not enough to protect the page?
15. What does `{% for poll in polls %}` do?
16. What does `{% for option in poll.options.all %}` do?
17. What is the purpose of `{% csrf_token %}`?
18. What is the difference between viewing a poll and voting in a poll?

---

## Final checklist

Before the next lesson, make sure that:

- [ ] `PollForm` has been added to `core/forms.py`.
- [ ] The polls list view has been created.
- [ ] The poll creation view has been created.
- [ ] Both URL patterns have been added to the existing URL configuration.
- [ ] `polls.html` has been created.
- [ ] `create_poll.html` has been created.
- [ ] The Polls link has been added to `base.html`.
- [ ] A moderator can create a poll with three options.
- [ ] Regular users cannot create polls.
- [ ] Unauthenticated users are redirected to log in.
- [ ] You have answered the self-check questions.

**Remember:** The goal is not just to make the website work. The goal is to understand how models, forms, views, URLs, templates, and permissions work together in Django.

If you get an error, read it carefully, find the line where it happens, and try to understand the reason. You can ask me for help if you get stuck. Good luck! 💻
