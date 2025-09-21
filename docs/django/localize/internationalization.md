let’s localize your Django project to **English (en)** and **Persian (fa)** step-by-step like a senior Django dev. I’ll cover the full flow (settings, marking strings, extracting & compiling messages, language switching, RTL handling, JS, model/content translations, tooling and CI). Follow the steps below and copy/paste the snippets into your project.

## 1) Quick overview (what you’ll do)

1. Enable i18n in `settings.py` and add languages + locale paths.
2. Add `LocaleMiddleware` (correct position).
3. Mark all user-facing strings for translation in templates and Python.
4. Run `makemessages` to generate `.po` files, translate them, then `compilemessages`.
5. Add a language switcher and (optionally) `i18n_patterns` for URL prefixes.
6. Handle RTL (Persian) in templates/CSS (Tailwind notes).
7. Translate JS strings and model/content (DB) where needed.
8. Add tooling (Poedit / Rosetta) and CI steps.

---

## 2) Settings — enable i18n & languages

Open `settings.py` and add/confirm these (adjust `BASE_DIR` as you have it):

```py
# settings.py
from pathlib import Path
from django.utils.translation import gettext_lazy as _

BASE_DIR = Path(__file__).resolve().parent.parent

USE_I18N = True

# default site language (fallback)
LANGUAGE_CODE = "en"

# Available languages in your UI
LANGUAGES = [
    ("en", _("English")),
    ("fa", _("فارسی")),   # show native name for better UX
]

# Where Django will look for locale files (project-level)
LOCALE_PATHS = [
    BASE_DIR / "locale",
]
```

Notes:

* Use `gettext_lazy` as `_` so language names can be localized.
* Keep `LANGUAGE_CODE` as your default (e.g. `"en"`). You can switch to `"fa"` if you want Persian as default.

---

## 3) Middleware order

Make sure `LocaleMiddleware` is **after** `SessionMiddleware` and **before** `CommonMiddleware`:

```py
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.locale.LocaleMiddleware",   # <- here
    "django.middleware.common.CommonMiddleware",
    # ... other middleware ...
]
```

---

## 4) Mark strings for translation

### In Python (views, models, forms)

Use `gettext_lazy` for class/module-level strings, `gettext` for runtime strings:

```py
# models.py
from django.db import models
from django.utils.translation import gettext_lazy as _

class Profile(models.Model):
    full_name = models.CharField(_("Full name"), max_length=255)
    bio = models.TextField(_("Biography"), blank=True)
```

For choices:

```py
STATUS_CHOICES = [
    ("draft", _("Draft")),
    ("published", _("Published")),
]
```

### In templates

At the top of templates where you use `{% trans %}`:

```django
{% load i18n %}

<h1>{% trans "Welcome back" %}</h1>

{% blocktrans with name=user.get_full_name %}
  Hello {{ name }}, welcome.
{% endblocktrans %}
```

`trans` for short strings, `blocktrans` when you embed variables or longer text.

---

## 5) Extract messages (makemessages) and compile

From your project root (where `manage.py` is), run:

* Create messages for all languages listed in `LANGUAGES`:

```bash
python manage.py makemessages -a
```

* Or create just Persian:

```bash
python manage.py makemessages -l fa
```

* If you use JavaScript and want JS catalogs:

```bash
python manage.py makemessages -d djangojs -l fa
```

After editing `.po` files, compile them into binary `.mo`:

```bash
python manage.py compilemessages
```

**Important:** `compilemessages` requires GNU gettext (the `msgfmt` tool).
If missing, install:

* Ubuntu/Debian: `sudo apt-get install gettext`
* macOS (Homebrew): `brew install gettext` and `brew link --force gettext` (if needed)
* Windows: use Chocolatey / install binaries (or edit `.po` with Poedit which can create `.mo`)

`makemessages` can be slow; exclude `node_modules`, venvs:

```bash
python manage.py makemessages -a --ignore venv --ignore node_modules
```

---

## 6) Where the `.po` files live

If `LOCALE_PATHS = [BASE_DIR / "locale"]`, you’ll see:

```
/locale/fa/LC_MESSAGES/django.po
/locale/en/LC_MESSAGES/django.po   # if you created it
```

You can also have `locale` dirs in each app — Django merges them.

Edit `.po` (manually or using a GUI like Poedit). Commit the `.po` files to VCS. Usually you do **not** commit compiled `.mo` (compile during build/CI).

---

## 7) Language switching (end-user)

Add Django’s i18n URLs and the `set_language` view.

`urls.py` (project):

```py
from django.urls import path, include
from django.conf.urls.i18n import i18n_patterns

urlpatterns = [
    path("i18n/", include("django.conf.urls.i18n")),  # exposes set_language view
]

urlpatterns += i18n_patterns(
    path("", include("myapp.urls")),
    prefix_default_language=False,   # optional: don't prefix default language
)
```

Language switcher template:

```django
{% load i18n %}
<form action="{% url 'set_language' %}" method="post">
  {% csrf_token %}
  <input name="next" type="hidden" value="{{ request.get_full_path }}">
  <select name="language">
    {% get_available_languages as LANGUAGES %}
    {% get_current_language as LANGUAGE_CODE %}
    {% for code, name in LANGUAGES %}
      <option value="{{ code }}" {% if code == LANGUAGE_CODE %}selected{% endif %}>{{ name }}</option>
    {% endfor %}
  </select>
  <button type="submit">{% trans "Change" %}</button>
</form>
```

`set_language` sets the cookie `django_language`. `LocaleMiddleware` will respect that on subsequent requests.

---

## 8) RTL (Persian) — HTML & CSS handling

For Persian you must flip layout direction to `rtl`. Use the language to set the `dir` attribute:

```django
{% load i18n %}
{% get_current_language as LANGUAGE_CODE %}
<html lang="{{ LANGUAGE_CODE }}" dir="{% if LANGUAGE_CODE == 'fa' %}rtl{% else %}ltr{% endif %}">
```

If you use Tailwind CSS:

* You can add a simple `dir` switch in templates and write styles that respond to `[dir="rtl"]` selectors.
* For a robust solution, consider an RTL plugin for Tailwind (e.g. `tailwindcss-rtl`) or building separate CSS for RTL. Many teams also apply a class like `class="rtl"` on `<html>` and use that to adapt components.

Example: flipping utilities in Tailwind without plugin may require custom CSS for margin/padding direction. Using a dedicated RTL plugin automates mirroring many utilities.

---

## 9) JavaScript translations

If you have strings in JS, expose translations with Django’s JS catalog:

`urls.py`:

```py
from django.views.i18n import JavaScriptCatalog

urlpatterns += [
    path('jsi18n/', JavaScriptCatalog.as_view(), name='javascript-catalog'),
]
```

In templates:

```html
<script src="{% url 'javascript-catalog' %}"></script>
<script>
  // now in JS you can use gettext("...") or ngettext(...)
  console.log(gettext("Hello world"));
</script>
```

Extract JS messages:

```bash
python manage.py makemessages -d djangojs -l fa
python manage.py compilemessages
```

---

## 10) Translating dynamic content (models / DB)

Django i18n handles UI strings, not model field values stored in DB. Options:

* If you need per-record translations (product descriptions, etc.), use a library:

  * `django-modeltranslation` — keep single DB table and add translated fields.
  * `django-parler` — separate translations table per model (good for many languages).
* If content is editorial, store separate records or fields with language metadata.

Pick and install the package you prefer and follow its docs. I can suggest exact setup for either if you want.

---

## 11) Helpful tools & workflow

* Edit `.po` with **Poedit** (desktop) or **django-rosetta** (web UI).

  * Rosetta: `pip install django-rosetta`, add `'rosetta'` to `INSTALLED_APPS`, and `path('rosetta/', include('rosetta.urls'))`.
* CI: add `python manage.py compilemessages` into CI/build so `.mo` files are produced on deploy.
* Keep `.po` files under source control. Don’t commit `.mo` if you compile at build time.
* For ambiguous strings, use `pgettext`/`pgettext_lazy` to provide context:
  `from django.utils.translation import pgettext_lazy as p_` -> `p_("button label", "Share")`

---

## 12) Testing & QA checklist

* [ ] Do `makemessages -a` and confirm `.po` files contain strings you expect.
* [ ] Edit `locale/fa/LC_MESSAGES/django.po` and add Persian translations.
* [ ] Run `compilemessages`.
* [ ] Start server and switch language (use switcher) — UI should show translated strings.
* [ ] Verify RTL layout and form inputs (numbers/dates) look right.
* [ ] Test JS strings and admin translations.
* [ ] If using caching, clear caches or account for translation caching.
* [ ] Add regression tests for a few translated pages (optional).

---

## 13) Short checklist for commands (cut & paste)

```bash
# 1. create po files for all languages configured
python manage.py makemessages -a

# OR create just fa
python manage.py makemessages -l fa

# 2. (if you have JS)
python manage.py makemessages -d djangojs -l fa

# 3. edit .po files in locale/fa/LC_MESSAGES/django.po (use Poedit or Rosetta)

# 4. compile
python manage.py compilemessages

# 5. restart the server if necessary
```

---

## 14) Gotchas & tips

* Mark strings everywhere: templates, Python, models, admin, forms. Missed strings remain untranslated.
* Use translation context (`pgettext`) for ambiguous words (e.g., “Post” noun vs verb).
* Keep translations in VCS; use CI to compile.
* Persian plural forms and plural rules are handled in `.po` headers — Poedit usually handles the header for you.
* For date formatting in Persian (Jalali), Django doesn’t provide it natively — consider `django-jalali` or transform dates in presentation layer.

---

If you want, I can now:

* Generate a **diff** for your `settings.py` + `urls.py` with the exact changes applied, **and**
* Provide a ready-to-drop **language switcher template** and a small **Tailwind RTL snippet** to auto-flip layout for `fa`.

