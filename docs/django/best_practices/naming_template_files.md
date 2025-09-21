There isn’t a *hard rule* enforced by Django for naming template files, but there **are strong community conventions and best practices** that Django developers follow for consistency, scalability, and maintainability.

---

## ✅ Standard Conventions for Django Template Naming

### 1. **Use lowercase with underscores**

* Similar to Python file naming (`snake_case`).
* Example:

  ```
  product_list.html
  product_detail.html
  user_profile.html
  ```

---

### 2. **Include the app name (namespacing)**

* To avoid collisions between apps, prefix templates with the app’s name (or keep them in an app-specific subdirectory).
* Example project structure:

  ```
  templates/
      shop/
          product_list.html
          product_detail.html
      accounts/
          login.html
          signup.html
  ```

This allows rendering with:

```python
return render(request, "shop/product_list.html")
```

---

### 3. **Follow Django’s view type convention**

* Common suffixes based on the view:

  * `list` → for list views (`product_list.html`)
  * `detail` → for detail views (`product_detail.html`)
  * `form` → for forms (`user_form.html` or `signup_form.html`)
  * `confirm_delete` → for delete confirmation views (`product_confirm_delete.html`)

This matches Django’s **generic class-based views** naming.

---

### 4. **Use `base` and inheritance**

* Standard to create a base layout:

  ```
  base.html
  base_auth.html
  ```
* Then extend it:

  ```django
  {% extends "base.html" %}
  ```

---

### 5. **Partial templates (reusable blocks)**

* Prefix with `_` to indicate they’re partials (not full pages).

  ```
  _navbar.html
  _form_fields.html
  _pagination.html
  ```

And include them:

```django
{% include "partials/_navbar.html" %}
```

---

### 6. **Keep consistency**

* Use plural for list views: `products_list.html`
* Singular for detail views: `product_detail.html`

---

✅ **Example structure:**

```
templates/
    base.html
    shop/
        product_list.html
        product_detail.html
        _product_card.html
    accounts/
        login.html
        signup.html
```

---

👉 So the **standard best practice**:

* `snake_case` filenames
* App-based directories for namespacing
* Use `list/detail/form/confirm_delete` suffixes aligned with Django’s generic views
* Use `_partial.html` for includes

---

Perfect 🙌 Let me give you a **standard, production-ready Django template folder structure** that scales well for medium-to-large projects.

This follows Django’s conventions + clean architecture practices.

---

## 📂 Recommended Template Folder Structure

```
project_root/
│
├── templates/                     # Global templates folder
│   ├── base/                      # Base layouts
│   │   ├── base.html              # Main site-wide layout
│   │   ├── base_auth.html         # For auth pages (login/signup)
│   │   ├── base_admin.html        # For admin/staff dashboards
│   │
│   ├── partials/                  # Reusable partials
│   │   ├── _navbar.html
│   │   ├── _footer.html
│   │   ├── _pagination.html
│   │   ├── _messages.html         # Flash messages
│   │   ├── _form_fields.html
│   │
│   ├── errors/                    # Error pages
│   │   ├── 403.html
│   │   ├── 404.html
│   │   ├── 500.html
│   │
│   ├── shop/                      # App-specific templates
│   │   ├── product_list.html
│   │   ├── product_detail.html
│   │   ├── product_form.html
│   │   ├── product_confirm_delete.html
│   │   ├── _product_card.html
│   │
│   ├── accounts/                  # App-specific templates
│   │   ├── login.html
│   │   ├── signup.html
│   │   ├── profile.html
│   │   ├── password_change_form.html
│   │   ├── password_reset_form.html
│   │
│   └── dashboard/                 # Another app (e.g. staff dashboard)
│       ├── dashboard_home.html
│       ├── reports.html
│       ├── stats.html
│
├── static/                        # Static files (CSS, JS, Images, etc.)
│   ├── css/
│   ├── js/
│   ├── images/
│
└── manage.py
```

---

## ✅ Best Practices in This Structure

1. **Base Layouts**

   * `base.html`: global layout (navbar, footer, container).
   * `base_auth.html`: minimal layout for login/register pages.
   * `base_admin.html`: separate look for dashboards.

2. **Partials**

   * Always prefix with `_` (like in Django and Flask conventions).
   * Makes it clear they are included templates:

     ```django
     {% include "partials/_navbar.html" %}
     ```

3. **Error Pages**

   * Custom `403.html`, `404.html`, `500.html` in `errors/`.
   * Django automatically looks for these if configured.

4. **App-Specific Namespacing**

   * Each app has its own folder under `templates/`.
   * This avoids collisions between apps (`shop/product_list.html` vs `accounts/user_list.html`).

5. **Generic View Compatibility**

   * Use suffixes: `_list.html`, `_detail.html`, `_form.html`, `_confirm_delete.html`.
   * Matches Django’s class-based views and reduces mental overhead.

6. **Consistency**

   * Always use `snake_case`.
   * Plural for lists (`products_list.html`), singular for details (`product_detail.html`).

---

⚡ Example: Rendering a shop product list

```python
# views.py
from django.shortcuts import render
from .models import Product

def product_list(request):
    products = Product.objects.all()
    return render(request, "shop/product_list.html", {"products": products})
```

```django
{# shop/product_list.html #}
{% extends "base/base.html" %}

{% block content %}
  <h1>Products</h1>
  <div class="product-grid">
    {% for product in products %}
      {% include "shop/_product_card.html" with product=product %}
    {% endfor %}
  </div>
{% endblock %}
```