# Group Panel — Homework Guide

## Delete Announcements and Build the Events Feature

This guide will help you finish the announcement functionality and then add a complete Events section to **Group Panel**.

You will work step by step. After each important stage, run the server and check the result **in your browser**.

> **Important:** We will use the terminal only to run the Django development server and create database migrations. We will check the website in the browser. You do not need to use Django Admin, Django shell, or automated tests for this homework.

---

## Table of contents

1. [Before you start](#1-before-you-start)
2. [Part A — Delete announcements](#2-part-a--delete-announcements)
   - [Step 1: Create the delete view](#step-1-create-the-delete-view)
   - [Step 2: Create the confirmation page](#step-2-create-the-confirmation-page)
   - [Step 3: Add the URL](#step-3-add-the-url)
   - [Step 4: Add the Delete link](#step-4-add-the-delete-link)
   - [Step 5: Test announcement deletion](#step-5-test-announcement-deletion)
3. [Part B — Create the Events feature](#3-part-b--create-the-events-feature)
   - [Step 1: Create the Event model](#step-1-create-the-event-model)
   - [Step 2: Create and run migrations](#step-2-create-and-run-migrations)
   - [Step 3: Create the Event form](#step-3-create-the-event-form)
   - [Step 4: Add the views](#step-4-add-the-views)
   - [Step 5: Add the URLs](#step-5-add-the-urls)
   - [Step 6: Create the Events list page](#step-6-create-the-events-list-page)
   - [Step 7: Create the Event details page](#step-7-create-the-event-details-page)
   - [Step 8: Create the Event form page](#step-8-create-the-event-form-page)
   - [Step 9: Create the Edit Event page](#step-9-create-the-edit-event-page)
   - [Step 10: Create the Delete Event page](#step-10-create-the-delete-event-page)
   - [Step 11: Add Events to the navigation](#step-11-add-events-to-the-navigation)
   - [Step 12: Test the complete Events feature](#step-12-test-the-complete-events-feature)
4. [Troubleshooting](#4-troubleshooting)
5. [Final checklist](#5-final-checklist)

---

# 1. Before you start

## Understand the current project

Your project already has an `Announcement` model and functionality for creating, viewing, and editing announcements. You have also created a helper function that checks whether a user can manage announcements.

The helper function should look similar to this:

```python
def can_manage_announcements(user):
    if user.is_superuser:
        return True

    is_moderator = user.groups.filter(name="Moderators").exists()
    return is_moderator
```

### What does this function do?

- `user.is_superuser` checks whether the user is a Django superuser.
- `user.groups.filter(name="Moderators").exists()` checks whether the user belongs to the group named `Moderators`.
- The function returns `True` for superusers and moderators, and `False` for other users.

We will reuse this function for Events too. Its name mentions announcements, but its current logic checks the user's role, so it can be used for both features. If you later rename it, remember to update every place where it is called.

## Start the server

1. Open your project in VS Code.
2. Open the terminal in the project folder — the folder that contains `manage.py`.
3. Run:

```bash
python manage.py runserver
```

4. Open the local address shown in the terminal, usually `http://127.0.0.1:8000/`.

Keep the terminal running while you work. If Django prints an error, read the last lines of the error message first.

> **When do you need the terminal?** You will use it to run the server and, later, to create and apply migrations. All feature checks will happen on the website.

---

# 2. Part A — Delete announcements

We will implement deletion in four places:

1. A view that checks permissions and deletes the selected announcement.
2. A confirmation template so the user does not delete something accidentally.
3. A URL that points to the view.
4. A link on the announcement details page.

We will use a **POST request** to perform the deletion. Opening a page with a link should not immediately delete data; the user must confirm the action by submitting a form.

## Step 1: Create the delete view

Open `core/views.py`.

Find the existing helper function `can_manage_announcements`. Then add this view below your existing announcement views. Do not delete or replace your other views.

```python
@login_required
def delete_announcement(request, announcement_id):
    # Only moderators and superusers can delete announcements.
    if not can_manage_announcements(request.user):
        return HttpResponse("Access denied.", status=403)

    # Find the announcement or return a 404 page if it does not exist.
    announcement = get_object_or_404(Announcement, id=announcement_id)

    # Delete only after the confirmation form has been submitted.
    if request.method == "POST":
        announcement.delete()
        return redirect("announcements")

    # For a GET request, show the confirmation page first.
    return render(
        request,
        "core/delete_announcement.html",
        {"announcement": announcement},
    )
```

### Understand the code

**`@login_required`**

Only logged-in users can open this view. Django redirects unauthenticated visitors to the login page.

**`request`**

Django passes the current HTTP request to the view. It contains information about what the visitor is doing.

**`announcement_id`**

The URL will contain the ID of the announcement to delete. For example, an address ending in `/announcements/5/delete/` passes the number `5` to the view.

**Permission check**

```python
if not can_manage_announcements(request.user):
    return HttpResponse("Access denied.", status=403)
```

This prevents ordinary users from deleting announcements. A `403` status means the server understood the request but refuses access.

**Find the announcement**

```python
announcement = get_object_or_404(Announcement, id=announcement_id)
```

Django looks for an announcement with that ID. If there is no such announcement, it returns a 404 page instead of causing an error.

**Delete only on POST**

```python
if request.method == "POST":
    announcement.delete()
    return redirect("announcements")
```

- `POST` means the form has been submitted.
- `.delete()` removes the database record.
- `redirect("announcements")` sends the visitor to the announcements list after deletion.

A `GET` request only displays the confirmation page. This is safer than deleting an object as soon as someone opens a URL.

**Render the template**

```python
return render(
    request,
    "core/delete_announcement.html",
    {"announcement": announcement},
)
```

This opens the confirmation page and passes the selected announcement to it. The template can then display its title.

### Check your imports

At the top of `core/views.py`, make sure you already have these imports. Add only the ones that are missing.

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.http import HttpResponse
from django.contrib.auth.decorators import login_required
from .models import Announcement
```

Do not add duplicate imports if they are already there.

### Checkpoint

Save `views.py`. The server may reload automatically, but the new feature is not ready to use yet because the template and URL do not exist.

---

## Step 2: Create the confirmation page

Create a new file:

`core/templates/core/delete_announcement.html`

Use this code:

```html
{% extends "core/base.html" %}

{% block content %}
    <h1>Delete announcement</h1>

    <p>
        Are you sure you want to delete
        <strong>{{ announcement.title }}</strong>?
    </p>

    <p>This action cannot be undone.</p>

    <form method="post">
        {% csrf_token %}
        <button type="submit">Yes, delete announcement</button>
    </form>

    <a href="{% url 'announcement_detail' announcement.id %}">
        Cancel
    </a>
{% endblock %}
```

### Understand the template

**`{% extends "core/base.html" %}`**

The page reuses the shared layout from `base.html`, including the site's navigation and other common elements.

**`{% block content %}` and `{% endblock %}`**

These mark the part of the base template where this page's content should appear.

**`{{ announcement.title }}`**

Django inserts the title of the announcement passed from the view.

**`<form method="post">`**

The form submits a POST request. The view will delete the announcement only when it receives this request.

**`{% csrf_token %}`**

Django's Cross-Site Request Forgery protection token. Include it in internal POST forms so Django can verify that the request was submitted through an expected form.

**The Cancel link**

```html
<a href="{% url 'announcement_detail' announcement.id %}">Cancel</a>
```

This uses the URL pattern named `announcement_detail` and passes the current announcement's ID. It takes the user back without deleting anything.

### Checkpoint

Save the file. The page still will not open until we add its URL.

---

## Step 3: Add the URL

Open the URL configuration where your announcement routes are defined. In this project, routes are connected in the project-level `group_portal/urls.py`.

Add this route to the existing `urlpatterns` list:

```python
path(
    "announcements/<int:announcement_id>/delete/",
    delete_announcement,
    name="delete_announcement",
),
```

Make sure `path` and `delete_announcement` are imported. Because the project currently uses `from core.views import *`, the view may already be available. Keep your project's existing import style for now.

### How does this URL work?

- `announcements/` is the section of the website.
- `<int:announcement_id>` captures an integer from the URL.
- `delete/` indicates the action.
- `delete_announcement` is the view that handles the request.
- `name="delete_announcement"` gives the route a name that templates can use.

For example, the URL for announcement ID `5` will be:

`http://127.0.0.1:8000/announcements/5/delete/`

The ID is only an example. Use an ID that exists in your database when testing.

### Checkpoint

Save the URL file. If the server reports an error, check that the view name is spelled the same way in both the import and the route.

---

## Step 4: Add the Delete link

Open:

`core/templates/core/announcement_detail.html`

Find the place where you added the **Edit** link. Add a Delete link nearby:

```html
<a href="{% url 'delete_announcement' announcement.id %}">
    Delete announcement
</a>
```

This link opens the confirmation page. It does **not** delete the announcement immediately.

### Checkpoint

Save the template and open an announcement's details page in the browser. You should see the Delete link if you added it to the correct template.

---

## Step 5: Test announcement deletion

Use the website for all checks.

### Test A — Open the confirmation page

1. Open the announcements list in your browser.
2. Open an announcement's details page.
3. Click **Delete announcement**.
4. Check that the confirmation page displays the correct announcement title.
5. Click **Cancel**.

**Expected result:** you return to the announcement details page, and the announcement still exists.

### Test B — Delete an announcement

1. Open the same announcement's details page again.
2. Click **Delete announcement**.
3. Check the title on the confirmation page.
4. Click **Yes, delete announcement**.

**Expected result:** you are redirected to the announcements list, and the deleted announcement no longer appears there.

### Test C — Check the details page

Try to open the deleted announcement's old details URL, if you know it.

**Expected result:** Django displays a 404 page because the announcement no longer exists.

### Test D — Check permissions

If possible, sign in as a regular user who is not a moderator or superuser.

1. Open an announcement's details page.
2. If the Delete link is visible, click it.
3. Observe the result.

**Expected result:** the view must deny access with a 403 response. Even if you hide the Delete link from ordinary users, the permission check in the view is still necessary.

> If your project currently displays the Delete link to everyone, that is not enough to grant permission. The server-side check in `delete_announcement` is what protects the action. You can improve link visibility later.

**Part A is complete when deletion works and only moderators/superusers can perform it.**

---

# 3. Part B — Create the Events feature

Now we will add a new section where group members can view upcoming events. Moderators and superusers will be able to create, edit, and delete events.

We will use the same general Django pattern you used for announcements:

**Model → Form → View → Template → URL → Browser check**

## Step 1: Create the Event model

Open `core/models.py`.

Keep your existing `Announcement` model. Add the `Event` model below it:

```python
from django.conf import settings
from django.db import models


class Event(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField()
    event_date = models.DateTimeField()
    location = models.CharField(max_length=200, blank=True)
    created_by = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )
    creation_date = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

If `from django.db import models` is already at the top of the file, do not add it twice. Add the `settings` import if it is missing.

### What does each field mean?

| Field | Purpose |
|---|---|
| `title` | The name of the event, such as “Project Presentation”. |
| `description` | A longer explanation of the event. |
| `event_date` | The date and time when the event will take place. |
| `location` | Where the event takes place. It can be left empty. |
| `created_by` | The Django user who created the event. |
| `creation_date` | The date and time when the record was created. |

### Understand the field types

**`models.CharField(max_length=200)`**

Stores a short piece of text. `max_length` sets its maximum length.

**`models.TextField()`**

Stores longer text, such as an event description.

**`models.DateTimeField()`**

Stores a date and time. We will use it for `event_date`.

**`blank=True`**

Allows the location field to be empty in a form. It does not mean that every field should be optional.

**`models.ForeignKey(...)`**

Connects each event to one Django user. This is better than saving the author's name as plain text because the event is linked to an actual account.

**`settings.AUTH_USER_MODEL`**

Refers to the user model configured for the project. This is more flexible than hard-coding Django's default user model.

**`on_delete=models.CASCADE`**

If the associated user account is deleted, Django will also delete events linked to that account. This is the rule we choose for this project.

**`auto_now_add=True`**

Django fills in `creation_date` automatically when a new event is created.

**`__str__`**

Controls how an Event object is represented as text in places where Django needs a readable name.

### Checkpoint

Save `models.py`. The model class is now defined in Python, but the database has not been updated yet.

---

## Step 2: Create and run migrations

A Django model describes the data structure in Python. **Migrations** tell Django how to apply that structure to the database.

In the terminal, stop the server with `Ctrl+C` if it is running. Make sure the terminal is in the folder containing `manage.py`.

Run:

```bash
python manage.py makemigrations
```

Django should create a migration that includes the new `Event` model.

Then run:

```bash
python manage.py migrate
```

This applies the migration to the database.

Start the server again:

```bash
python manage.py runserver
```

Open the website in your browser.

### Checkpoint

The website should load without errors. We have not created the Events pages yet, so you may not see an Events link at this point.

If Django reports that a field or model is missing, check `models.py` and the migration output before continuing.

---

## Step 3: Create the Event form

Create a new file:

`core/forms.py`

If `forms.py` already exists and contains `AnnouncementForm`, **do not replace the file**. Add the new form below the existing code instead.

```python
from django import forms
from .models import Announcement, Event


class EventForm(forms.ModelForm):
    event_date = forms.DateTimeField(
        input_formats=["%Y-%m-%dT%H:%M"],
        widget=forms.DateTimeInput(
            format="%Y-%m-%dT%H:%M",
            attrs={"type": "datetime-local"},
        ),
    )

    class Meta:
        model = Event
        fields = ["title", "description", "event_date", "location"]
```

If `Announcement` is not needed in this file or your existing imports differ, preserve your current code and add only the `Event` import and `EventForm` class.

### Understand the form

**`forms.ModelForm`**

A ModelForm creates form fields based on a model. It helps us validate input and save a database object without manually writing every field.

**`model = Event`**

Tells Django which model this form works with.

**`fields = [...]`**

Lists the fields the visitor can enter. We do not include `created_by` or `creation_date`:

- `created_by` will be set from the logged-in user in the view.
- `creation_date` is filled in automatically by Django.

**The `event_date` field**

The browser's `datetime-local` input provides a date and time in a format like `2026-11-15T14:30`. The `format` and `input_formats` settings tell Django how to display and read that value.

### Checkpoint

Save `forms.py`. The form is ready to be used by the views.

---

## Step 4: Add the views

Open `core/views.py`.

Keep all your existing announcement views. Add the `Event` and `EventForm` imports if they are not already present:

```python
from .models import Announcement, Event
from .forms import AnnouncementForm, EventForm
```

If `Announcement` or `AnnouncementForm` is already imported, update the existing import instead of duplicating it.

Make sure you also have these imports. Add only missing ones:

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.http import HttpResponse
from django.contrib.auth.decorators import login_required
```

Now add the following views **below your existing views**.

### 4.1 View all events

```python
def events(request):
    all_events = Event.objects.order_by("event_date")

    return render(
        request,
        "core/events.html",
        {"events": all_events},
    )
```

**Explanation**

- `Event.objects` lets us query the Event database table.
- `.order_by("event_date")` sorts events by their date, with earlier events first.
- `{"events": all_events}` passes the list to the template under the name `events`.

This page is available for viewing. Creating, editing, and deleting will have separate permission checks.

### 4.2 View one event

```python
def event_detail(request, event_id):
    event = get_object_or_404(Event, id=event_id)

    return render(
        request,
        "core/event_detail.html",
        {"event": event},
    )
```

This view finds one event by its ID and displays its details. If the ID does not exist, Django returns a 404 page.

### 4.3 Create an event

```python
@login_required
def create_event(request):
    if not can_manage_announcements(request.user):
        return HttpResponse("Access denied.", status=403)

    if request.method == "POST":
        form = EventForm(request.POST)

        if form.is_valid():
            event = form.save(commit=False)
            event.created_by = request.user
            event.save()

            return redirect("event_detail", event_id=event.id)
    else:
        form = EventForm()

    return render(
        request,
        "core/event_form.html",
        {"form": form},
    )
```

**Why do we use `request.method`?**

- A `GET` request opens the page and displays an empty form.
- A `POST` request happens when the visitor submits the form.

**`form.is_valid()`**

Django checks the submitted values against the form's rules. If the data is invalid, the form is displayed again with error messages.

**`form.save(commit=False)`**

This creates an Event object from the form data but does not save it to the database yet.

We use this because `created_by` is not a field the visitor should choose. We set it ourselves:

```python
event.created_by = request.user
event.save()
```

This ensures that the event is linked to the account that is currently signed in.

**`redirect("event_detail", event_id=event.id)`**

After saving, the browser opens the new event's details page. Redirecting after a successful POST also prevents an accidental duplicate submission if the page is refreshed.

### 4.4 Edit an event

```python
@login_required
def edit_event(request, event_id):
    if not can_manage_announcements(request.user):
        return HttpResponse("Access denied.", status=403)

    event = get_object_or_404(Event, id=event_id)

    if request.method == "POST":
        form = EventForm(request.POST, instance=event)

        if form.is_valid():
            form.save()
            return redirect("event_detail", event_id=event.id)
    else:
        form = EventForm(instance=event)

    return render(
        request,
        "core/edit_event.html",
        {"form": form, "event": event},
    )
```

**What is `instance=event`?**

It tells Django that the form should work with an existing Event object.

- When the page is opened with GET, `EventForm(instance=event)` fills the fields with the current values.
- When the form is submitted, `EventForm(request.POST, instance=event)` validates the new values and updates that same event.

Without `instance=event`, Django would treat the submitted data as a new event rather than editing the existing one.

### 4.5 Delete an event

```python
@login_required
def delete_event(request, event_id):
    if not can_manage_announcements(request.user):
        return HttpResponse("Access denied.", status=403)

    event = get_object_or_404(Event, id=event_id)

    if request.method == "POST":
        event.delete()
        return redirect("events")

    return render(
        request,
        "core/delete_event.html",
        {"event": event},
    )
```

This follows the same safe pattern as announcement deletion:

- GET displays a confirmation page.
- POST deletes the event.
- After deletion, the visitor returns to the Events list.
- Only moderators and superusers can perform the action.

### Checkpoint

Save `views.py`. The views are defined, but the pages and URL patterns still need to be created.

---

## Step 5: Add the URLs

Open the URL configuration in `group_portal/urls.py`.

Make sure `path` is imported:

```python
from django.urls import path
```

Your project currently imports views with `from core.views import *`. If you keep that style, the view functions should already be available. Add these routes to the existing `urlpatterns` list, keeping all existing routes:

```python
path("events/", events, name="events"),
path("events/create/", create_event, name="create_event"),
path("events/<int:event_id>/", event_detail, name="event_detail"),
path("events/<int:event_id>/edit/", edit_event, name="edit_event"),
path("events/<int:event_id>/delete/", delete_event, name="delete_event"),
```

### Why does the order matter?

Django checks URL patterns in order. Put the fixed path `events/create/` before the general path `events/<int:event_id>/`. Otherwise, Django may try to interpret `create` as an event ID, depending on the route definitions.

The integer routes use the event's ID. For example, `/events/3/` opens the event with ID `3`.

### Checkpoint

Save the URL file. Open this address in your browser:

`http://127.0.0.1:8000/events/`

If the view and URL are connected correctly, Django will try to render `core/events.html`. We have not created that template yet, so a template error at this stage is expected.

---

## Step 6: Create the Events list page

Create:

`core/templates/core/events.html`

```html
{% extends "core/base.html" %}

{% block content %}
    <h1>Group Events</h1>

    <p>
        <a href="{% url 'create_event' %}">Create a new event</a>
    </p>

    {% for event in events %}
        <article>
            <h2>
                <a href="{% url 'event_detail' event.id %}">
                    {{ event.title }}
                </a>
            </h2>

            <p><strong>Date:</strong> {{ event.event_date }}</p>

            {% if event.location %}
                <p><strong>Location:</strong> {{ event.location }}</p>
            {% endif %}

            <p>{{ event.description|truncatewords:25 }}</p>
        </article>
        <hr>
    {% empty %}
        <p>There are no events yet.</p>
    {% endfor %}
{% endblock %}
```

The Create link is visible to everyone in this first version to keep the template simple. A regular user who clicks it will receive **Access denied** from the view, because the view checks permissions on the server. Later, you can improve the navigation so the link is displayed only to moderators and superusers.

Remember: hiding a link is a user-interface choice. The permission check inside the view is what protects the action.

### Understand the list template

- `{% for event in events %}` loops through all events passed by the view.
- `event.title`, `event.event_date`, `event.location`, and `event.description` display the fields of the current event.
- `{% if event.location %}` hides the location paragraph if no location was entered.
- `truncatewords:25` displays up to 25 words of the description on the list page.
- `{% empty %}` displays a message when the list contains no events.
- The title links to the event details page.

### Checkpoint — test the list page

1. Save `events.html`.
2. Open `http://127.0.0.1:8000/events/` in your browser.
3. Check that the heading **Group Events** appears.
4. If there are no events yet, check that **There are no events yet** appears.

If the page loads, the list view, URL, and template are connected.

---

## Step 7: Create the Event details page

Create:

`core/templates/core/event_detail.html`

```html
{% extends "core/base.html" %}

{% block content %}
    <h1>{{ event.title }}</h1>

    <p><strong>Date and time:</strong> {{ event.event_date }}</p>

    {% if event.location %}
        <p><strong>Location:</strong> {{ event.location }}</p>
    {% endif %}

    <h2>About this event</h2>
    <p>{{ event.description|linebreaks }}</p>

    <p>
        <a href="{% url 'events' %}">Back to all events</a>
    </p>

    <p>
        <a href="{% url 'edit_event' event.id %}">Edit event</a>
    </p>

    <p>
        <a href="{% url 'delete_event' event.id %}">Delete event</a>
    </p>
{% endblock %}
```

### Understand the template

- `{{ event.title }}` displays the event's title.
- `{{ event.event_date }}` displays the saved date and time.
- `linebreaks` turns line breaks in the description into paragraphs and line breaks in HTML.
- The Edit and Delete links include the current event ID.

These links are shown here for simplicity. The views still check permissions, so ordinary users cannot edit or delete events. Later, you can hide the links from users who do not have permission.

### Checkpoint

Save the file. You cannot open a real event yet until you create one, so continue to the form page.

---

## Step 8: Create the Event form page

Create:

`core/templates/core/event_form.html`

```html
{% extends "core/base.html" %}

{% block content %}
    <h1>Create an event</h1>

    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}

        <button type="submit">Create event</button>
    </form>

    <p>
        <a href="{% url 'events' %}">Cancel</a>
    </p>
{% endblock %}
```

### Understand the template

- `method="post"` submits the form data to the current view.
- `{% csrf_token %}` enables Django's CSRF protection for the form.
- `{{ form.as_p }}` renders each Django form field inside a paragraph, including validation errors if there are any.
- The button submits the form.
- Cancel returns to the Events list without creating an event.

### Checkpoint — create your first event

1. Sign in as a moderator or superuser.
2. Open `http://127.0.0.1:8000/events/`.
3. Click **Create a new event**.
4. Enter a title, description, date and time. Location is optional.
5. Click **Create event**.

**Expected result:** you are redirected to the new event's details page, and the event information is displayed.

If the event does not save, check the validation errors shown by the form. Make sure you selected a valid date and time.

---

## Step 9: Create the Edit Event page

Create:

`core/templates/core/edit_event.html`

```html
{% extends "core/base.html" %}

{% block content %}
    <h1>Edit event</h1>

    <p>
        You are editing: <strong>{{ event.title }}</strong>
    </p>

    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}

        <button type="submit">Save changes</button>
    </form>

    <p>
        <a href="{% url 'event_detail' event.id %}">Cancel</a>
    </p>
{% endblock %}
```

### Checkpoint — test editing

1. Open the Events list.
2. Click the event you created.
3. Click **Edit event**.
4. Check that the form is already filled with the event's current values.
5. Change the title or description.
6. Click **Save changes**.

**Expected result:** the browser returns to the event details page, and the updated values appear.

If the form is empty, check that both calls to `EventForm` in `edit_event` use `instance=event`.

---

## Step 10: Create the Delete Event page

Create:

`core/templates/core/delete_event.html`

```html
{% extends "core/base.html" %}

{% block content %}
    <h1>Delete event</h1>

    <p>
        Are you sure you want to delete
        <strong>{{ event.title }}</strong>?
    </p>

    <p>This action cannot be undone.</p>

    <form method="post">
        {% csrf_token %}
        <button type="submit">Yes, delete event</button>
    </form>

    <p>
        <a href="{% url 'event_detail' event.id %}">Cancel</a>
    </p>
{% endblock %}
```

### Checkpoint — test deletion

1. Open the event details page.
2. Click **Delete event**.
3. Check that the confirmation page shows the correct event title.
4. Click **Cancel** and verify that the event still exists.
5. Open the confirmation page again and click **Yes, delete event**.

**Expected result:** you return to the Events list, and the deleted event no longer appears.

---

## Step 11: Add Events to the navigation

Open:

`core/templates/core/base.html`

Find the navigation links, such as Home and Announcements. Add this link next to them:

```html
<a href="{% url 'events' %}">Events</a>
```

Use the same surrounding HTML and CSS classes as your existing navigation links so the new link matches the site's design.

### Checkpoint

Save the file and refresh the website. The **Events** link should appear in the navigation. Click it and confirm that the Events list opens.

---

## Step 12: Test the complete Events feature

Use this checklist in your browser.

### Viewing

- [ ] Click **Events** in the navigation.
- [ ] Confirm that the Events list opens.
- [ ] Click an event title.
- [ ] Confirm that the event details page opens.
- [ ] Use **Back to all events** to return to the list.

### Creating

- [ ] Sign in as a moderator or superuser.
- [ ] Open the Create Event page.
- [ ] Create an event with a title, description, date and time.
- [ ] Leave the location empty and check that the form still works.
- [ ] Confirm that the new event appears in the list.

### Editing

- [ ] Open an event.
- [ ] Click **Edit event**.
- [ ] Confirm that the existing data appears in the form.
- [ ] Change at least one field and save.
- [ ] Confirm that the updated value appears on the details page and list.

### Deleting

- [ ] Open an event.
- [ ] Click **Delete event**.
- [ ] Click **Cancel** and confirm the event still exists.
- [ ] Delete the event through the confirmation form.
- [ ] Confirm that it disappears from the list.

### Permissions

- [ ] Sign in as a regular user who is not a moderator or superuser.
- [ ] Try to open `/events/create/`.
- [ ] If you know an event ID, try `/events/ID/edit/` and `/events/ID/delete/`, replacing `ID` with the actual number.
- [ ] Confirm that the views return **Access denied** for those actions.
- [ ] Confirm that regular users can still view the Events list and event details.

**Important:** the links in templates are for navigation, not security. The permission checks inside the views are what prevent unauthorized changes.

---

# 4. Troubleshooting

## `TemplateDoesNotExist`

**What it means:** Django cannot find the template named in the view.

**Check:**
- The file is inside `core/templates/core/`.
- The filename is spelled correctly.
- The view uses the same template path, for example `"core/events.html"`.

## `NoReverseMatch`

**What it means:** Django could not find a URL with the name used in a template or redirect.

**Check:**
- The URL name is spelled exactly the same in `urls.py` and in `{% url ... %}`.
- Routes that require an ID receive the correct argument, such as `event.id` or `event_id=event.id`.

## `NameError: Event is not defined`

**What it means:** `views.py` does not know what `Event` refers to.

**Check:** make sure `Event` is imported from `.models`.

## `NameError: EventForm is not defined`

**Check:** make sure `EventForm` is imported from `.forms`.

## `no such table: core_event`

**What it means:** the database table for Event has not been created.

**Fix:** stop the server and run:

```bash
python manage.py makemigrations
python manage.py migrate
```

Then start the server again.

## `403 Access denied`

This may be expected if you are signed in as a regular user. Only moderators and superusers can create, edit, or delete events.

If you are a moderator but still receive a 403 response, check:
- You are signed in to the correct account.
- The account belongs to the Django group named exactly `Moderators`, or is a superuser.
- The permission helper checks the same group name.

## The date/time form shows an error

Check that:
- You selected both a date and a time.
- `EventForm` has the `datetime-local` widget and matching input format shown in this guide.
- The `event_date` field in the model is a `DateTimeField`.

## The server shows a Python error

1. Read the last lines of the terminal output.
2. Find the file and line number mentioned in the error.
3. Check spelling, indentation, imports and brackets around that line.
4. Save the file and refresh the browser.

Do not immediately delete large sections of code. Try to identify the specific cause first.

---

# 5. Final checklist

Before you finish, make sure that:

- [ ] Only moderators and superusers can delete announcements.
- [ ] Announcement deletion requires confirmation.
- [ ] Deleted announcements disappear from the list.
- [ ] The Event model exists and migrations have been applied.
- [ ] The Events list page opens.
- [ ] A visitor can open an event details page.
- [ ] Moderators and superusers can create events.
- [ ] Moderators and superusers can edit events.
- [ ] Moderators and superusers can delete events.
- [ ] Regular users cannot create, edit, or delete events.
- [ ] The Events link appears in the navigation.
- [ ] All checks above have been performed through the website.

## A final reminder

Work in small steps. Save your files, check for errors, and test each feature before moving on. If something does not work, read the error message and investigate it rather than rewriting the whole project.

You do not need to make everything perfect on the first attempt. The goal is to understand how models, forms, views, templates, URLs, database migrations, and permissions work together in Django.
