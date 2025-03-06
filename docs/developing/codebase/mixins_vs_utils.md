# Mixins vs. Utils: Key Differences

Mixins and utility functions serve different purposes in Django development, and understanding when to use each is
important for maintaining a well-structured codebase.

### Mixins

1. **Class-based reusability**: Mixins are classes designed to be inherited from to provide common functionality to
   other classes.

2. **Used with class-based views**: Primarily used to extend Django's class-based views with additional functionality.

3. **Tightly coupled to class hierarchy**: Mixins participate in method resolution order (MRO) and can override or
   enhance methods of parent classes.

4. **Stateful**: Can maintain state across method calls within the instance.

5. **Example use cases**:
    - Authentication and permissions
    - Form handling
    - Context data preparation
    - Response formatting

### Utils (Utility Functions)

1. **Function-based reusability**: Standalone functions that perform specific tasks.

2. **Context-independent**: Can be used anywhere, not limited to classes.

3. **Loosely coupled**: No inheritance involved; simply called with arguments.

4. **Stateless**: Typically pure functions that produce the same output for the same input.

5. **Example use cases**:
    - Data formatting and transformation
    - Calculations and conversions
    - File operations
    - Common validations

## Examples of Each

### Mixin Example:

```python
class TitleContextMixin:
    page_title = None

    def get_page_title(self):
        return self.page_title

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['page_title'] = self.get_page_title()
        return context


class ProductListView(TitleContextMixin, ListView):
    model = Product
    page_title = "Our Products"
    # The view now automatically adds page_title to the context
```

### Utility Function Example:

```python
# In utils.py
def generate_product_slug(product_name):
    """Generate a URL-friendly slug from a product name"""
    slug = product_name.lower().replace(' ', '-')
    return re.sub(r'[^a-z0-9-]', '', slug)


# In views.py or models.py
from .utils import generate_product_slug


def create_product(name, price):
    slug = generate_product_slug(name)
    return Product.objects.create(name=name, price=price, slug=slug)
```

## When to Use Which

### Use Mixins When:

- Extending class-based views
- Reusing code that needs to interact with the class's methods
- You need to override specific parts of a class's behavior
- The functionality is specific to a certain type of class

### Use Utility Functions When:

- The functionality is generic and widely applicable
- Logic doesn't depend on class state
- The code should be usable in any context (views, models, forms, etc.)
- Performing simple operations that don't require inheritance

## Best Practices

### For Mixins:

1. Keep mixins focused on a single responsibility
2. Name mixins clearly with a descriptive suffix ("Mixin")
3. Document method overrides and expected parent class methods
4. Be careful of the inheritance order (MRO)
5. Place in a dedicated `mixins.py` file

### For Utilities:

1. Group related utilities in modules (e.g., `text_utils.py`, `date_utils.py`)
2. Write thorough docstrings explaining parameters and return values
3. Make utilities as pure as possible (avoid side effects)
4. Write comprehensive tests for utilities
5. Place in a dedicated `utils.py` file or a `utils/` package

With a clear understanding of these differences, you can make more informed decisions about code organization in your
Django projects, leading to more maintainable and readable code.