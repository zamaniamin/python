In Django, you should generally use `gettext_lazy` in these cases:

## Use `gettext_lazy` for:

1. **Models**:
    - Field definitions (verbose_name, help_text)
    - Model Meta options (verbose_name, verbose_name_plural)
    - Choices for fields

2. **Forms**:
    - Field labels, help_text
    - Error messages
    - Form Meta options

3. **Class-based Views**:
    - Any class attributes that contain translatable text
    - Page titles, success messages, etc.

4. **Admin**:
    - Any text in ModelAdmin classes

5. **URL patterns**:
    - Any translatable text in URL pattern names

The general rule is: use `gettext_lazy` for any string that is defined at module level or as a class attribute.

## Example for a Class-based View:

```python
from django.utils.translation import gettext_lazy as _
from django.views.generic import CreateView
from .models import Product


class ProductCreateView(CreateView):
    model = Product
    template_name = 'products/create.html'
    success_message = _("Product created successfully")
    page_title = _("Create New Product")
```

## Example for a Form:

```python
from django.utils.translation import gettext_lazy as _
from django import forms
from .models import Product


class ProductForm(forms.ModelForm):
    extra_info = forms.CharField(
        label=_("Additional Information"),
        help_text=_("Any extra details about the product"),
        required=False
    )

    class Meta:
        model = Product
        fields = ['name', 'status']
        labels = {
            'name': _("Product Name"),
            'status': _("Product Status"),
        }
        help_texts = {
            'status': _("Choose the current status of this product"),
        }
        error_messages = {
            'name': {
                'required': _("Product name cannot be empty"),
                'max_length': _("Product name is too long"),
            }
        }
```

## When to use regular `gettext` instead:

Use regular `gettext` (not lazy) inside functions and methods where the translation should happen immediately, in the
context of the current request:

```python
from django.utils.translation import gettext as _


class ProductView(View):
    def get(self, request, *args, **kwargs):
        # Here we use gettext because we're inside a method
        # and want to translate immediately using the current request's language
        messages.success(request, _("Product viewed successfully"))
        return render(request, 'product.html')
```

The key difference is that `gettext_lazy` delays translation until the string is actually rendered (which is what you
want for class attributes), while `gettext` translates immediately (which is what you want inside methods where you have
access to the current user's language).