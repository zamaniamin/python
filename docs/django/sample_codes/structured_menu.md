# Sample Code for Structured Menu

Create a structured way to represent menu data in Python and create a template tag to render it. Here's how you can
approach this:

First, let's define a data structure to represent your menu:

```python
# Example menu data structure
menu_data = {
    'title': 'Title',
    'items': [
        {'text': 'Item 1', 'icon': 'home', 'link': '/home/'},
        {'text': 'Item 2', 'icon': 'user', 'link': '/profile/'},
        {'text': 'Item 3', 'icon': 'settings', 'link': '/settings/'},
    ],
    'css_class': 'menu bg-base-200 rounded-box w-56'
}
```

Now, let's create a template tag to render this menu. You'll need to create a file structure like this:

```
your_app/
    templatetags/
        __init__.py
        menu_tags.py
```

Here's the implementation for the `menu_tags.py` file:

```python
from django import template
from django.utils.safestring import mark_safe
from django.template.loader import render_to_string

register = template.Library()


@register.simple_tag
def render_menu(menu_data):
    """
    Renders a menu based on the provided menu data.
    
    Expected menu_data format:
    {
        'title': 'Menu Title',
        'items': [
            {'text': 'Item Text', 'icon': 'icon-name', 'link': '/url/'},
            ...
        ],
        'css_class': 'optional-css-classes'
    }
    """
    return mark_safe(render_to_string('components/menu.html', {'menu': menu_data}))
```

Then create a template file at `templates/components/menu.html`:

```html

<ul class="{{ menu.css_class|default:'menu bg-base-200 rounded-box w-56' }}">
    {% if menu.title %}
    <li class="menu-title">{{ menu.title }}</li>
    {% endif %}
    {% for item in menu.items %}
    <li>
        <a href="{{ item.link }}">
            {% if item.icon %}
            <i class="icon-{{ item.icon }}"></i>
            {% endif %}
            {{ item.text }}
        </a>
    </li>
    {% endfor %}
</ul>
```

To use this in your templates:

```html
{% load menu_tags %}

{# Using a single menu #}
{% render_menu menu_data %}

{# Or if you have multiple menus #}
{% for menu in menus %}
{% render_menu menu %}
{% endfor %}
```

For a more dynamic approach, you could also create a context processor to make menus available globally:

```python
# In your app's context_processors.py
def menus(request):
    # This could come from a database or settings
    menus = [
        {
            'title': 'Main Menu',
            'items': [
                {'text': 'Home', 'icon': 'home', 'link': '/'},
                {'text': 'Profile', 'icon': 'user', 'link': '/profile/'},
            ],
        },
        {
            'title': 'User Menu',
            'items': [
                {'text': 'Settings', 'icon': 'cog', 'link': '/settings/'},
                {'text': 'Logout', 'icon': 'logout', 'link': '/logout/'},
            ],
        }
    ]
    return {'menus': menus}
```

Then add this to your settings:

```python
TEMPLATES = [
    {
        # ...
        'OPTIONS': {
            'context_processors': [
                # ...
                'your_app.context_processors.menus',
            ],
        },
    },
]
```

This approach gives you a clean, reusable way to define menus in Python and render them consistently across your
templates.