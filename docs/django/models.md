# Models And Databases

I'll demonstrate this principle using an e-commerce website example that shows how to add custom methods to [Django
models](https://docs.djangoproject.com/en/dev/topics/db/models/) to encapsulate business logic.

## Describe this by good example of e-commerce website

Define custom methods on a model to add custom “row-level” functionality to your objects. Whereas `Manager` methods are
intended to do “table-wide” things, model methods should act on a particular model instance.
This is a valuable technique for keeping business logic in one place – the model."

```python title="product model"
from django.db import models
from django.core.exceptions import ValidationError
from django.utils import timezone
from decimal import Decimal


class Product(models.Model):
    """Product model with custom business logic methods"""
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.PositiveIntegerField(default=0)
    discount_rate = models.DecimalField(
        max_digits=5,
        decimal_places=2,
        default=Decimal('0.00')
    )
    created_at = models.DateTimeField(auto_now_add=True)

    def apply_discount(self, percentage):
        """
        Apply a discount to the product
        
        This method demonstrates a row-level method that modifies 
        the instance's pricing logic
        """
        if not (0 <= percentage <= 100):
            raise ValidationError("Discount must be between 0 and 100 percent")

        self.discount_rate = Decimal(percentage) / 100
        self.save()
        return self.discounted_price

    @property
    def discounted_price(self):
        """
        Calculate the current price after applying discount
        
        This is a computed property that uses the model's own data
        """
        return self.price * (1 - self.discount_rate)

    def is_in_stock(self):
        """
        Check if the product is currently available
        
        A simple method that provides business logic at the model level
        """
        return self.stock > 0

    def reduce_stock(self, quantity):
        """
        Reduce product stock and prevent overselling
        
        Demonstrates encapsulating inventory management logic
        """
        if quantity > self.stock:
            raise ValidationError(f"Not enough stock. Only {self.stock} available.")

        self.stock -= quantity
        self.save()
        return self.stock

```

```python title="order model"
class Order(models.Model):
    """Order model with custom business logic methods"""
    user = models.ForeignKey('auth.User', on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    is_paid = models.BooleanField(default=False)
    total_price = models.DecimalField(max_digits=10, decimal_places=2, default=Decimal('0.00'))

    def calculate_total(self):
        """
        Calculate the total price of all order items
        
        A method that computes aggregate information for the order
        """
        total = sum(item.total_price for item in self.orderitem_set.all())
        self.total_price = total
        self.save()
        return total

    def mark_as_paid(self):
        """
        Mark the order as paid and process payment-related logic
        
        Demonstrates encapsulating payment state changes
        """
        if self.is_paid:
            raise ValidationError("Order is already paid")

        self.is_paid = True
        self.save()
        return self.is_paid

    def can_be_cancelled(self):
        """
        Determine if an order can be cancelled
        
        Business logic for order cancellation rules
        """
        # Can only cancel unpaid orders created less than 24 hours ago
        return (not self.is_paid and
                timezone.now() - self.created_at < timezone.timedelta(hours=24))

```

```python title="order item model"
class OrderItem(models.Model):
    """Order Item model with custom methods"""
    order = models.ForeignKey(Order, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField()

    @property
    def total_price(self):
        """
        Calculate the total price for this order item
        
        A computed property that uses product and quantity
        """
        return self.product.discounted_price * self.quantity

    def validate_stock(self):
        """
        Validate that there's enough product stock
        
        Business logic for ensuring order can be fulfilled
        """
        if self.quantity > self.product.stock:
            raise ValidationError(f"Insufficient stock for {self.product.name}")
        return True

```

```python title="Example usage demonstration"
def process_order(user, product, quantity):
    """
    Demonstrate how these custom model methods work together
    """
    # Create an order
    order = Order.objects.create(user=user)

    # Create an order item
    order_item = OrderItem.objects.create(
        order=order,
        product=product,
        quantity=quantity
    )

    # Use model methods to validate and process
    try:
        # Validate stock before processing
        order_item.validate_stock()

        # Reduce product stock
        product.reduce_stock(quantity)

        # Calculate order total
        order.calculate_total()

        # Mark as paid
        order.mark_as_paid()

        return order
    except ValidationError as e:
        # Handle any business logic validation errors
        print(f"Order processing failed: {e}")
        return None

```

## **Break down the key principles demonstrated in this examples**

### 1. **Row-Level Functionality**

Each model has methods that operate on a specific instance:

- `Product.apply_discount()`: Applies a discount to a specific product
- `Product.reduce_stock()`: Manages inventory for a specific product
- `Order.mark_as_paid()`: Changes the state of a specific order
- `OrderItem.validate_stock()`: Checks stock for a specific order item

### 2. **Encapsulation of Business Logic**

- Business rules are contained within the models themselves
- Methods handle validation, state changes, and calculations
- Prevents spreading business logic across multiple files

### 3. **Computed Properties**

- `discounted_price`: Calculates price dynamically
- `total_price`: Computes total based on current state

### 4. **Validation and Error Handling**

- Custom methods include validation logic
- Raises meaningful exceptions when business rules are violated

### 5. **Separation of Concerns**

- Model methods focus on instance-specific behaviors
- Keeps related logic close to the data it operates on

## **Benefits of this Approach**

- Centralized business logic
- Easier to maintain and understand
- Prevents duplicating logic across views or services
- Makes the code more testable
- Leverages Django's ORM capabilities

## **Real-World Implications**

In an e-commerce context, these methods allow you to:

- Apply discounts dynamically
- Manage inventory
- Handle order processing
- Implement business rules directly in the model layer

## **When to Use**

- Complex domain logic
- Calculations that depend on model attributes
- State changes with specific business rules
- Validations that go beyond simple field constraints

This approach aligns with the principle of keeping business logic in one place – the model – making your code more
organized, readable, and maintainable.

---

Now provide a comprehensive explanation of Django Model Managers, demonstrating their use in the e-commerce context and
contrasting them with model methods.

```python title="product manager"
from django.db import models
from django.db.models import Q, Sum, Avg
from django.utils import timezone
from decimal import Decimal


class ProductManager(models.Manager):
    """
    Custom Manager for Product model
    Handles "table-wide" operations and complex queries
    """

    def active_products(self):
        """
        Returns only products that are currently available
        Table-wide query to filter products
        """
        return self.filter(
            stock__gt=0,
            is_active=True
        )

    def discounted_products(self, min_discount=0.1):
        """
        Find products with significant discounts
        """
        return self.filter(
            discount_rate__gte=min_discount
        )

    def top_selling_products(self, limit=10):
        """
        Find most sold products
        Requires additional logic with OrderItem
        """
        from django.db.models import Count

        return self.annotate(
            total_sales=Count('orderitem')
        ).order_by('-total_sales')[:limit]

    def create_featured_product(self, name, price, stock):
        """
        Custom method to create a product with additional logic
        """
        product = self.create(
            name=name,
            price=price,
            stock=stock,
            is_featured=True
        )
        return product


class Product(models.Model):
    """Product model with custom manager"""
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.PositiveIntegerField(default=0)
    is_active = models.BooleanField(default=True)
    is_featured = models.BooleanField(default=False)
    discount_rate = models.DecimalField(
        max_digits=5,
        decimal_places=2,
        default=Decimal('0.00')
    )

    # Replace the default manager with our custom manager
    objects = ProductManager()

    # Additional model methods (as in previous example)
    def apply_discount(self, percentage):
        """Instance-level discount method"""
        self.discount_rate = Decimal(percentage) / 100
        self.save()
        return self

```

```python title="order manager"
class OrderManager(models.Manager):
    """
    Custom Manager for Order model
    Provides complex querying and aggregation methods
    """

    def recent_orders(self, days=30):
        """
        Retrieve orders from the last specified number of days
        """
        threshold = timezone.now() - timezone.timedelta(days=days)
        return self.filter(created_at__gte=threshold)

    def total_revenue(self, start_date=None, end_date=None):
        """
        Calculate total revenue within a specific date range
        """
        queryset = self.all()

        if start_date:
            queryset = queryset.filter(created_at__gte=start_date)

        if end_date:
            queryset = queryset.filter(created_at__lte=end_date)

        return queryset.aggregate(
            total_revenue=Sum('total_price'),
            avg_order_value=Avg('total_price')
        )

    def user_order_summary(self, user):
        """
        Generate order summary for a specific user
        """
        return {
            'total_orders': self.filter(user=user).count(),
            'total_spent': self.filter(user=user).aggregate(
                total=Sum('total_price')
            )['total'] or Decimal('0.00'),
            'last_order_date': self.filter(user=user).latest('created_at').created_at if self.filter(
                user=user).exists() else None
        }


class Order(models.Model):
    """Order model with custom manager"""
    user = models.ForeignKey('auth.User', on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    total_price = models.DecimalField(max_digits=10, decimal_places=2)

    # Replace default manager with custom manager
    objects = OrderManager()

```

```python title="example usage demonstration"
def demonstrate_managers():
    """
    Demonstrate the use of custom managers
    """
    # Finding active products
    featured_products = Product.objects.active_products()

    # Creating a featured product
    new_product = Product.objects.create_featured_product(
        name="Super Cool Product",
        price=99.99,
        stock=100
    )

    # Finding discounted products
    discounted = Product.objects.discounted_products(min_discount=0.2)

    # Retrieving recent orders
    recent_orders = Order.objects.recent_orders(days=7)

    # Calculating total revenue
    revenue_stats = Order.objects.total_revenue(
        start_date=timezone.now() - timezone.timedelta(days=30),
        end_date=timezone.now()
    )

    return {
        'featured_products': featured_products,
        'new_product': new_product,
        'discounted_products': discounted,
        'recent_orders': recent_orders,
        'revenue_stats': revenue_stats
    }

```

## **Key Differences between Model Methods and Managers**

### **Scope of Operation**

- **Model Methods**:
    - Operate on a specific model instance
    - Modify or interact with a single object's state
    - Example: `product.apply_discount()`

- **Managers**:
    - Operate at the table/queryset level
    - Perform queries, filtering, and aggregations
    - Example: `Product.objects.active_products()`

### **When to Use Each**

**Use Model Methods When**:

- You need to perform actions on a specific instance
- The logic is specific to a single object
- You're modifying the state of an individual record

**Use Managers When**:

- You want to perform queries across multiple objects
- You need complex filtering or aggregation
- You want to add table-wide query methods

### **Key Capabilities of Managers**

- Custom querysets
- Complex filtering
- Aggregation methods
- Factory methods for creating objects
- Encapsulating query logic

## **Practical Examples**

```python
# Model Method (Instance-level)
single_product = Product.objects.get(id=1)
single_product.apply_discount(20)  # Works on this specific product

# Manager Method (Table-level)
Product.objects.discounted_products()  # Returns all discounted products
Product.objects.top_selling_products()  # Returns most sold products
```

## **Best Practices**

1. Use default `objects` manager sparingly
2. Create custom managers for complex query logic
3. Keep managers focused on querying and retrieval
4. Use model methods for instance-specific behaviors
5. Combine managers and methods for comprehensive model behavior

## **Common Use Cases for Managers**

- Filtering active/inactive records
- Generating reports
- Complex search functionality
- Aggregating data
- Custom object creation with additional logic

## **Limitations to Remember**

- Managers are class-level, not instance-level
- They should primarily focus on retrieving and creating objects
- Complex business logic is better handled in model methods or services

---

## [Relationships](https://docs.djangoproject.com/en/5.1/topics/db/models/#relationships)

* [Many-to-one relationships](https://docs.djangoproject.com/en/5.1/topics/db/models/#many-to-one-relationships)
* [Many-to-many relationships](https://docs.djangoproject.com/en/5.1/topics/db/models/#many-to-many-relationships)
* [One-to-one relationships](https://docs.djangoproject.com/en/5.1/topics/db/models/#one-to-one-relationships)

---
source: [DjangoProject](https://docs.djangoproject.com)