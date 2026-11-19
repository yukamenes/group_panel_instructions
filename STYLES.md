# Homework: Start Styling the Group Panel Website with Bootstrap
 
## Goal
 
In this homework, you will begin improving the appearance of your
**Group Panel** Django website using **Bootstrap 5**.
 
The project already includes Announcements, Events, Forum, and Polls
with voting. Your task is to improve the HTML appearance without
rewriting the Django functionality.
 
By the end, you should have Bootstrap connected, a styled navigation
bar, a consistent page layout, and announcement, event, and poll content
displayed in Bootstrap cards.
 
**Important:** Keep your existing file names, Django URL names, context
variables, model field names, permission checks, and form logic. The
examples below show the structure and Bootstrap classes. If your project
uses different names, keep the names from your own code.
 
------------------------------------------------------------------------
 
# Part 1. What is Bootstrap?
 
Bootstrap is a front-end framework. It provides ready-made CSS styles
and interface components so developers do not need to write every style
from scratch.
 
For example:
 
``` html
<button>Save</button>
```
 
is a normal HTML button. It has no special Bootstrap appearance.
 
``` html
<button class="btn btn-primary">Save</button>
```
 
The classes `btn` and `btn-primary` apply Bootstrap's button styling.
 
A few useful Bootstrap classes:
 
| Class | Purpose |
|---|---|
| `container` | Creates a centred responsive content area |
| `row` | Creates a grid row |
| `col-12` | Uses the full width of a grid row |
| `mt-4` | Adds top margin |
| `mb-3` | Adds bottom margin |
| `p-3` | Adds padding on all sides |
| `card` | Creates a card component |
| `btn btn-primary` | Styles an element as a primary button |
| `text-muted` | Makes secondary text less prominent |
 
Bootstrap also helps create responsive websites. A responsive website
adapts to different screen sizes, such as desktop, tablet, and mobile.
 
## Checkpoint 1
 
1.  What is Bootstrap?
2.  What is the difference between HTML and CSS?
3.  What does a responsive website do?
------------------------------------------------------------------------
 
# Part 2. Connect Bootstrap to Django
 
## 2.1. Find `base.html`
 
Open the shared template used by the other pages. In our project, it
should be similar to:
 
``` text
core/
└── templates/
    └── core/
        └── base.html
```
 
Your exact folder structure may differ. Find the `base.html` that
contains the common HTML structure and navigation bar.
 
A base template prevents us from repeating the same HTML on every page.
Other templates can inherit from it using:
 
``` django
{% extends "core/base.html" %}
```
 
This tells Django to use the structure from `base.html` and insert the
child template's content into the appropriate block.
 
## 2.2. Add Bootstrap CSS
 
Inside the `<head>` element of `base.html`, add:
 
``` html
<link
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
    rel="stylesheet"
>
```
 
Your document should have a structure similar to this:
 
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
 
    <title>Group Panel</title>
 
    <link
        href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
        rel="stylesheet"
    >
</head>
<body>
 
    <!-- Navigation and page content will be here -->
 
</body>
</html>
```
 
### Explanation, line by line
 
-   `<!DOCTYPE html>` tells the browser to interpret the document as
    modern HTML.
-   `<html lang="en">` is the root HTML element. The `lang` attribute
    describes the main language of the page.
-   `<meta charset="UTF-8">` sets the character encoding. UTF-8 supports
    many languages and special characters.
-   `<meta name="viewport" content="width=device-width, initial-scale=1.0">`
    helps the browser display the page correctly on mobile devices.
    `width=device-width` matches the page width to the device width;
    `initial-scale=1.0` sets the initial zoom.
-   `<title>` sets the text displayed in the browser tab.
-   The Bootstrap `<link>` loads Bootstrap's CSS stylesheet from a CDN.
    A CDN is a server that distributes files, such as CSS and JavaScript
    libraries, over the internet.
-   `rel="stylesheet"` tells the browser that the linked file is a
    stylesheet.
You do not need to download Bootstrap manually for this homework. The
browser loads it from the CDN, so an internet connection is required.
 
## 2.3. Add Bootstrap JavaScript
 
Some Bootstrap components need JavaScript. For example, the navigation
menu will use JavaScript to open and close on smaller screens.
 
Add this line immediately before the closing `</body>` tag:
 
``` html
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
```
 
The `bootstrap.bundle.min.js` file contains Bootstrap's JavaScript and
the Popper library used by some components.
 
The CSS link belongs inside `<head>`. The script can be placed near the
end of `<body>`, after the page content.
 
## Checkpoint 2
 
1.  Where should the Bootstrap CSS link be placed?
2.  Where should the Bootstrap JavaScript script be placed?
3.  Why does a responsive navigation menu need JavaScript?
4.  What is the purpose of `base.html`?
------------------------------------------------------------------------
 
# Part 3. Improve the Main Page Layout
 
## 3.1. Use the `container` class
 
Without a layout container, page content may stretch across the entire
browser window and touch its edges.
 
Bootstrap's `container` class creates a centred content area with
responsive width and horizontal spacing.
 
Example:
 
``` html
<div class="container">
    <h1>Welcome to Group Panel</h1>
    <p>Manage announcements, events, discussions, and polls.</p>
</div>
```
 
In `base.html`, find the existing content block. Put it inside a
`<main>` element with Bootstrap classes. For example:
 
``` django
<main class="container my-4">
    {% block content %}
    {% endblock %}
</main>
```
 
If you already have a `{% block content %}`, do not create a second one.
Move the existing block inside `<main>`.
 
### Explanation
 
-   `<main>` represents the main content of the page. It gives the HTML
    document a clear semantic structure.
-   `container` applies Bootstrap's responsive content width and
    horizontal spacing.
-   `my-4` adds vertical margin. In Bootstrap spacing utilities, `m`
    means margin, `y` means the vertical direction (top and bottom), and
    `4` is the spacing size.
-   `{% block content %}` and `{% endblock %}` are Django template tags.
    They define the area where a child template inserts its content.
## 3.2. Style the page heading
 
On the home page, add or update a heading using a structure like this:
 
``` html
<div class="mb-4">
    <h1 class="display-5">Welcome to Group Panel</h1>
    <p class="text-muted">
        Your place for announcements, events, discussions, and polls.
    </p>
</div>
```
 
Explanation:
 
-   `mb-4` adds margin below the heading section. `m` means margin, `b`
    means bottom, and `4` is the spacing size.
-   `display-5` applies a large display heading style.
-   `text-muted` makes the paragraph less visually prominent, which is
    useful for supporting information.
### Your task
 
1.  Find the home page template.
2.  Keep its existing Django template inheritance and links.
3.  Add a clear page heading.
4.  Make sure the page content is inside the `container`.
5.  Use spacing classes to make the content easier to read.
Do not delete existing functionality.
 
------------------------------------------------------------------------
 
# Part 4. Style the Navigation Bar
 
The navigation bar is shared by multiple pages, so it belongs in
`base.html`.
 
Find your existing navigation HTML and update it. Do not add a second
navigation bar.
 
Use this example as a guide:
 
``` django
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container">
 
        <a class="navbar-brand" href="{% url 'home' %}">
            Group Panel
        </a>
 
        <button
            class="navbar-toggler"
            type="button"
            data-bs-toggle="collapse"
            data-bs-target="#mainNavbar"
            aria-controls="mainNavbar"
            aria-expanded="false"
            aria-label="Toggle navigation"
        >
            <span class="navbar-toggler-icon"></span>
        </button>
 
        <div class="collapse navbar-collapse" id="mainNavbar">
 
            <ul class="navbar-nav me-auto mb-2 mb-lg-0">
 
                <li class="nav-item">
                    <a class="nav-link" href="{% url 'announcements' %}">
                        Announcements
                    </a>
                </li>
 
                <li class="nav-item">
                    <a class="nav-link" href="{% url 'events' %}">
                        Events
                    </a>
                </li>
 
                <li class="nav-item">
                    <a class="nav-link" href="{% url 'forum' %}">
                        Forum
                    </a>
                </li>
 
                <li class="nav-item">
                    <a class="nav-link" href="{% url 'polls' %}">
                        Polls
                    </a>
                </li>
 
            </ul>
 
            <div class="d-flex">
                {% if user.is_authenticated %}
                    <span class="navbar-text me-3">
                        Hello, {{ user.username }}!
                    </span>
 
                    <a class="btn btn-outline-light btn-sm"
                       href="{% url 'logout' %}">
                        Log out
                    </a>
                {% else %}
                    <a class="btn btn-outline-light btn-sm"
                       href="{% url 'login' %}">
                        Log in
                    </a>
                {% endif %}
            </div>
 
        </div>
    </div>
</nav>
```
 
**Important:** This is a structural example. Your project may use
different URL names, such as `index`, `home_page`, or `forum_topics`.
The `{% url %}` tag must use a URL name that exists in your URL
configuration. Keep your actual names.
 
## 4.1. Bootstrap navigation classes
 
-   `navbar` activates Bootstrap's navigation component.
-   `navbar-expand-lg` makes the navigation horizontal on large screens
    and collapsible on smaller screens.
-   `navbar-dark` uses light text suitable for a dark background.
-   `bg-dark` gives the navigation bar a dark background.
-   `container` aligns the navigation content with the main page
    content.
-   `navbar-brand` styles the website name or logo.
-   `navbar-toggler` styles the button that opens and closes the menu on
    small screens.
-   `collapse navbar-collapse` makes the menu collapsible.
-   `navbar-nav` styles the list of navigation links.
-   `nav-item` identifies one navigation item.
-   `nav-link` styles a navigation link.
-   `me-auto` adds automatic margin at the end of the element in the
    current writing direction. Here, it pushes the account controls to
    the opposite side.
-   `mb-2 mb-lg-0` adds bottom margin on smaller screens and removes it
    on large screens.
-   `d-flex` applies a flexible layout to the child elements.
-   `btn btn-outline-light btn-sm` creates a small button with a light
    outline.
## 4.2. Django template code in the navigation bar
 
Bootstrap controls the appearance, but Django still generates dynamic
links and user information.
 
-   `{% url 'home' %}` asks Django to find the URL pattern named `home`
    and generate its URL.
-   `{% if user.is_authenticated %}` checks whether the current user is
    logged in.
-   `{{ user.username }}` displays the current user's username.
-   `{% else %}` defines what should be displayed if the user is not
    logged in.
-   `{% endif %}` ends the conditional block.
The navigation bar must continue to show the correct account links
depending on the user's authentication status.
 
## Checkpoint 3
 
1.  Why should the navigation bar be in `base.html`?
2.  What does `navbar-expand-lg` do?
3.  What does `data-bs-target="#mainNavbar"` connect?
4.  What is the difference between `nav-link` and `btn`?
5.  Why must you keep the URL names from your own project?
6.  What does `user.is_authenticated` check?
------------------------------------------------------------------------
 
# Part 5. Display Announcements in Bootstrap Cards
 
A card groups related content inside a visually separate area. Cards are
useful for announcements because each announcement has its own title,
content, author, and date.
 
Open the template that displays the announcement list.
 
Keep your existing Django loop and context variable. For example, if
your view sends `announcements`, the loop may look like:
 
``` django
{% for announcement in announcements %}
    <!-- One announcement -->
{% endfor %}
```
 
Update the HTML inside the loop using a structure similar to this:
 
``` django
<div class="row">
    {% for announcement in announcements %}
 
        <div class="col-12 mb-3">
            <div class="card shadow-sm">
 
                <div class="card-body">
 
                    <h5 class="card-title">
                        {{ announcement.title }}
                    </h5>
 
                    <p class="card-text">
                        {{ announcement.content }}
                    </p>
 
                    <p class="text-muted small mb-2">
                        Author: {{ announcement.author_name }}
                    </p>
 
                    <p class="text-muted small">
                        {{ announcement.creation_date }}
                    </p>
 
                    <a href="{% url 'announcement_detail' announcement.id %}"
                       class="btn btn-primary">
                        Read more
                    </a>
 
                </div>
            </div>
        </div>
 
    {% empty %}
        <p class="text-muted">There are no announcements yet.</p>
    {% endfor %}
</div>
```
 
Check your existing URL configuration and template before copying this
example. Your detail URL name or context variable may be different.
Preserve your project's names.
 
## 5.1. Bootstrap classes explained
 
-   `row` creates a grid row. Bootstrap columns are placed inside rows.
-   `col-12` makes the column use the full width of the row.
-   `mb-3` adds bottom margin, separating one card from the next.
-   `card` activates the Bootstrap card component.
-   `shadow-sm` adds a small shadow around the card.
-   `card-body` adds internal spacing around the card's content.
-   `card-title` styles the card heading.
-   `card-text` styles the main text.
-   `small` makes secondary text smaller.
-   `text-muted` makes secondary information less prominent.
-   `btn btn-primary` makes the link look like a primary button.
An `<a>` element is still a link, even when Bootstrap makes it look like
a button. It navigates to another page.
 
## 5.2. Django loop explained
 
-   `{% for announcement in announcements %}` repeats the HTML inside
    the loop for each announcement.
-   `{{ announcement.title }}` displays the current announcement's
    title.
-   `{{ announcement.content }}` displays its content.
-   `{{ announcement.author_name }}` displays the author's name.
-   `{{ announcement.creation_date }}` displays its creation date.
-   `{% empty %}` displays a message if there are no announcements.
-   `{% endfor %}` ends the loop.
Bootstrap does not replace the Django loop. Django supplies the data;
Bootstrap changes how the resulting HTML looks.
 
### Your task
 
1.  Find the announcement list template.
2.  Identify the existing loop.
3.  Keep the existing loop variable and model field names.
4.  Add the Bootstrap card structure around each announcement.
5.  Keep the existing detail, edit, or delete links and permission
    checks.
6.  Add an empty-state message if your template does not already have
    one.
------------------------------------------------------------------------
 
# Part 6. Display Events in Bootstrap Cards
 
Events should have a consistent appearance with announcements.
 
Open the events list template. Keep the existing loop and field names.
The following is an example:
 
``` django
<div class="row">
    {% for event in events %}
 
        <div class="col-12 col-md-6 mb-3">
            <div class="card h-100 shadow-sm">
 
                <div class="card-body">
 
                    <h5 class="card-title">
                        {{ event.title }}
                    </h5>
 
                    <p class="card-text">
                        {{ event.description }}
                    </p>
 
                    <p class="text-muted small">
                        Date: {{ event.date }}
                    </p>
 
                    <p class="text-muted small">
                        Location: {{ event.location }}
                    </p>
 
                </div>
            </div>
        </div>
 
    {% empty %}
        <p class="text-muted">There are no events yet.</p>
    {% endfor %}
</div>
```
 
The field names above are examples. Your `Event` model may use different
names, such as `event_date`, or may not have a `location` field. Use the
exact fields from your model and existing template.
 
## 6.1. New Bootstrap classes
 
-   `col-12` means the card uses the full row width on extra-small
    screens.
-   `col-md-6` means the card uses 6 of the 12 grid columns from the
    medium breakpoint upwards. Two cards can therefore fit next to each
    other on wider screens.
-   `h-100` sets the card height to 100% of the available parent height.
    It can help cards in the same row have a consistent height.
### Your task
 
1.  Find the events list template.
2.  Keep your existing loop and variables.
3.  Wrap each event in a Bootstrap card.
4.  Use `col-12 col-md-6` for a responsive two-column layout on wider
    screens.
5.  Keep existing event details and links.
6.  Make sure the empty list case still works.
------------------------------------------------------------------------
 
# Part 7. Display Polls in Bootstrap Cards
 
The Polls page should also have a clear structure. Each poll can be
displayed inside a Bootstrap card.
 
Open your polls template. This example assumes that:
 
-   Your context variable is named `polls`.
-   A poll has an `options` relationship.
-   The option text is stored in `option.text`.
-   Your voting view reads the selected option using
    `request.POST.get("option")`.
-   The voting URL is named `vote_poll`.
Your actual names may differ. Keep the names from your working
implementation.
 
``` django
<div class="row">
    {% for poll in polls %}
 
        <div class="col-12 mb-4">
            <div class="card shadow-sm">
 
                <div class="card-body">
 
                    <h5 class="card-title">
                        {{ poll.question }}
                    </h5>
 
                    {% if poll.is_active %}
 
                        <form method="POST"
                              action="{% url 'vote_poll' poll.id %}">
 
                            {% csrf_token %}
 
                            {% for option in poll.options.all %}
 
                                <div class="form-check mb-2">
 
                                    <input
                                        class="form-check-input"
                                        type="radio"
                                        name="option"
                                        value="{{ option.id }}"
                                        id="option{{ option.id }}"
                                        required
                                    >
 
                                    <label
                                        class="form-check-label"
                                        for="option{{ option.id }}"
                                    >
                                        {{ option.text }}
                                    </label>
 
                                </div>
 
                            {% endfor %}
 
                            <button type="submit" class="btn btn-success mt-3">
                                Vote
                            </button>
 
                        </form>
 
                    {% else %}
 
                        <p class="text-muted">
                            This poll is closed.
                        </p>
 
                    {% endif %}
 
                </div>
            </div>
        </div>
 
    {% empty %}
        <p class="text-muted">There are no polls yet.</p>
    {% endfor %}
</div>
```
 
**Important:** This is a styling example, not a replacement for your
existing voting implementation. Keep your existing conditions, URL name,
context variables, and model fields. Do not remove any logic that
prevents voting in inactive polls or handles users who have already
voted.
 
## 7.1. Form classes explained
 
-   `form-check` is the Bootstrap wrapper for a checkbox or radio input
    and its label.
-   `form-check-input` styles the radio input.
-   `form-check-label` styles the associated label.
-   `mb-2` adds bottom margin between options.
-   `btn btn-success` styles the submit button with Bootstrap's success
    colour.
-   `mt-3` adds margin above the button.
## 7.2. Why must radio buttons share the same `name`?
 
All options for one question must have the same `name` attribute:
 
``` html
<input type="radio" name="option" value="1">
<input type="radio" name="option" value="2">
<input type="radio" name="option" value="3">
```
 
The browser treats these inputs as one radio group, so the user can
select only one answer from the group.
 
The `value` identifies the selected option. When the form is submitted,
the browser sends that value to Django.
 
Your view may read it like this:
 
``` python
option_id = request.POST.get("option")
```
 
The string `"option"` in the HTML `name` must match the key used by the
Django view. If you change one without changing the other, Django will
not find the selected value under the expected key.
 
## 7.3. Why do we need `{% csrf_token %}`?
 
Django uses CSRF protection to help prevent other websites from
submitting forms on behalf of a logged-in user.
 
For a POST form handled by Django, include:
 
``` django
{% csrf_token %}
```
 
Do not remove it while styling the form.
 
### Your task
 
1.  Find the polls template.
2.  Keep the form's `method="POST"`.
3.  Keep `{% csrf_token %}`.
4.  Keep the radio input `name` expected by your view.
5.  Keep the selected option's ID in the `value`.
6.  Add Bootstrap card and form classes.
7.  Test that voting still works.
------------------------------------------------------------------------
 
# Part 8. Style Buttons and Forms
 
## 8.1. Bootstrap buttons
 
Examples:
 
``` html
<button class="btn btn-primary">Primary</button>
<button class="btn btn-success">Success</button>
<button class="btn btn-danger">Delete</button>
<button class="btn btn-warning">Warning</button>
<button class="btn btn-secondary">Secondary</button>
```
 
The `btn` class activates Bootstrap button styling. The second class
defines the button's colour and visual meaning.
 
| Class | Typical use |
|---|---|
| `btn-primary` | Main action |
| `btn-success` | Positive or successful action |
| `btn-danger` | Delete or dangerous action |
| `btn-warning` | Warning or action needing attention |
| `btn-secondary` | Secondary action |
| `btn-outline-primary` | Primary button with an outline |
 
For example, a link to create an announcement might look like this:
 
``` django
<a href="{% url 'announcement_create' %}" class="btn btn-primary">
    Create announcement
</a>
```
 
Keep your actual URL name.
 
## 8.2. Bootstrap form controls
 
A manually written input can use these classes:
 
``` html
<label for="title" class="form-label">Title</label>
 
<input
    type="text"
    id="title"
    name="title"
    class="form-control"
>
```
 
-   `form-label` styles the label.
-   `form-control` styles the input and makes it fit the available
    width.
If your Django forms use `{{ form.as_p }}`, adding Bootstrap classes may
require updating form widgets in `forms.py`. Do not change your forms
unless you understand how they are currently rendered. For this first
task, focus on styling the surrounding HTML and existing buttons.
 
------------------------------------------------------------------------
 
# Part 9. Test the Website
 
Run the development server from the folder containing `manage.py`:
 
``` bash
python manage.py runserver
```
 
Open your website in the browser and test every page you changed.
 
## Testing checklist
 
-   [ ] The home page opens.
-   [ ] Bootstrap styles are visible.
-   [ ] The navigation bar appears on pages that extend `base.html`.
-   [ ] Navigation links open the correct pages.
-   [ ] The navigation menu collapses on a narrow screen.
-   [ ] The logged-in user's name is displayed correctly.
-   [ ] Login and logout links still work.
-   [ ] Announcements are displayed in cards.
-   [ ] Announcement detail links still work.
-   [ ] Events are displayed in cards.
-   [ ] Polls are displayed in cards.
-   [ ] A user can select a poll option.
-   [ ] A user can submit a vote.
-   [ ] Existing permissions still work.
-   [ ] No Django template errors appear.
-   [ ] Pages remain usable when a list is empty.
## Check the responsive layout
 
1.  Open the website.
2.  Press `F12` to open Developer Tools.
3.  Find the device toolbar button (often called **Toggle device
    toolbar**).
4.  Select a mobile device size or make the viewport narrower.
5.  Check that the navigation menu and cards fit the screen.
You can also make the browser window narrower manually.
 
------------------------------------------------------------------------
 
# Part 10. Common Mistakes and How to Fix Them
 
## Mistake 1: Bootstrap CSS is in the wrong place
 
The Bootstrap stylesheet link belongs inside `<head>`. If it is placed
outside the document's head, the browser may not load it as expected.
 
## Mistake 2: Bootstrap JavaScript is missing
 
The navigation bar may look correct, but its mobile toggle button may
not work. Check that the Bootstrap bundle script is included before
`</body>`.
 
## Mistake 3: A URL name does not exist
 
For example:
 
``` django
<a href="{% url 'announcement_detail' announcement.id %}">
```
 
will fail if the URL configuration does not contain a pattern named
`announcement_detail`.
 
Use the exact URL name from your project.
 
## Mistake 4: Django variable names have changed
 
If your view sends a context variable named `events`, but your template
uses:
 
``` django
{% for event in all_events %}
```
 
the template will not receive the expected collection. Keep the context
variable names from your existing code.
 
## Mistake 5: CSRF protection was removed
 
Do not remove `{% csrf_token %}` from POST forms.
 
## Mistake 6: The poll input name was changed
 
If your view uses:
 
``` python
request.POST.get("option")
```
 
your radio inputs must use:
 
``` html
name="option"
```
 
Otherwise, the view will not find the selected value under the expected
key.
 
## Mistake 7: A working template was replaced completely
 
Do not delete the whole template and paste an example without checking
it. Your current template may contain permission checks, edit/delete
links, messages, forms, or other important Django code.
 
Keep that functionality and add Bootstrap classes around it.
 
## Mistake 8: Bootstrap classes and Django template tags are confused
 
Bootstrap classes look like this:
 
``` html
class="container mt-4"
class="card shadow-sm"
class="btn btn-primary"
```
 
Django template tags look like this:
 
``` django
{% for event in events %}
{% if user.is_authenticated %}
{% url 'events' %}
{% csrf_token %}
```
 
Bootstrap styles the HTML. Django generates dynamic content and handles
server-side functionality. They have different responsibilities and work
together.
 
------------------------------------------------------------------------
 
# Final Homework Requirements
 
Complete the following tasks:
 
1.  Connect Bootstrap CSS and JavaScript in `base.html`.
2.  Improve the main layout using `container` and spacing classes.
3.  Update the existing navigation bar using Bootstrap classes.
4.  Display announcements in Bootstrap cards.
5.  Display events in Bootstrap cards.
6.  Display polls in Bootstrap cards.
7.  Add Bootstrap styling to the buttons you encounter.
8.  Test the website and make sure the existing functionality still
    works.
## What to send me
 
When you finish:
 
1.  Send screenshots of:
    -   The home page
    -   The navigation bar
    -   The announcements page
    -   The events page
    -   The polls page
2.  Send your updated `base.html`.
3.  Send the templates you changed.
4.  Tell me if you encountered any errors and how you tried to solve
    them.
## Questions to Check Your Understanding
 
Try to answer these without copying the explanations above.
 
1.  What is Bootstrap, and what problem does it solve?
2.  What is the difference between `container` and `row`?
3.  What does `col-md-6` mean?
4.  What is the purpose of `card-body`?
5.  Why do we use `mb-3` or `mt-4`?
6.  What is the difference between `btn-primary` and `btn-success`?
7.  Why do we add Bootstrap CSS to `base.html` instead of adding it
    separately to every page?
8.  Does Bootstrap replace Django template tags? Explain your answer.
9.  Why must the radio input's `name` match the key expected by the
    voting view?
10. What could happen if you change a Django URL name in a template
    without changing the URL configuration?
**Remember:** The goal is not just to make the website look different.
You should understand what each Bootstrap class does and be able to
explain how the updated HTML continues to work with your Django code.
 
