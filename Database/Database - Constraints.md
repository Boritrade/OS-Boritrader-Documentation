# Managing Unique and Check Constraints in Django ORM
**The Boritrader Database Documentation**  
**Boritrade, LLC**  
**Updated Oct 21, 2024**  

This section provides a detailed overview of handling unique and check constraints in Django ORM. It covers best practices, code examples, and recommendations to ensure consistent application behavior and data integrity.

---

## **1. Unique Constraints in Django ORM**

In Django models, the `unique_together` constraint ensures that a combination of fields remains unique across records, preventing duplicate entries.

### **Defining Unique Constraints**

The following example enforces a unique `(user, algorithm)` combination:

```python
class Meta:
    unique_together = (('user', 'algorithm'),)
```

**Behavior:**  
- Attempting to insert a duplicate `(user, algorithm)` combination triggers an `IntegrityError`, ensuring data consistency.

---

### **Examples of Handling Unique Constraints**

#### **1. Successful Configuration Creation**

```python
user = User.objects.get(username='johndoe')
algorithm = Algorithm.objects.get(display_name='Mean Reversion')

UserAlgorithmSetting.objects.create(
    user=user,
    algorithm=algorithm,
    tickers=['BTC', 'ETH'],
    quantity=1.5,
    execution_frequency='1 hour'
)
```

**Outcome:**  
- A new entry is created since the combination `(user, algorithm)` does not yet exist.

#### **2. Handling Duplicate Configurations**

```python
from django.db import IntegrityError

try:
    UserAlgorithmSetting.objects.create(
        user=user,
        algorithm=algorithm,
        tickers=['BTC', 'LTC'],
        quantity=2.0,
        execution_frequency='30 minutes'
    )
except IntegrityError as e:
    print(f"IntegrityError: {e}")
```

**Outcome:**  
- An `IntegrityError` is raised because the combination `(user, algorithm)` already exists.

#### **3. Updating Existing Configurations**

Instead of inserting duplicate entries, use `update_or_create()` to either update an existing configuration or create a new one.

```python
setting, created = UserAlgorithmSetting.objects.update_or_create(
    user=user,
    algorithm=algorithm,
    defaults={
        'tickers': ['BTC', 'LTC'],
        'quantity': 2.0,
        'execution_frequency': '30 minutes'
    }
)
```

**Outcome:**  
- If the configuration exists, it is updated; otherwise, a new entry is created.

---

## **2. Check Constraints in Django ORM**

`CheckConstraint` enforces data rules at the database level, preventing invalid entries. These constraints ensure that business rules are applied even if application-level validations are bypassed.

### **Example: Check Constraints on `TradingPreferences` Model**

```python
class TradingPreferences(models.Model):
    user = models.OneToOneField(
        User, on_delete=models.CASCADE, primary_key=True, related_name='trading_preferences'
    )
    auto_purchase_enabled = models.BooleanField(default=False)
    notification_only_enabled = models.BooleanField(default=True)
    paper_trading_enabled = models.BooleanField(default=True)
    live_trading_enabled = models.BooleanField(default=False)

    class Meta:
        constraints = [
            models.CheckConstraint(
                check=~(models.Q(auto_purchase_enabled=True) & models.Q(notification_only_enabled=True)),
                name='auto_vs_notification_check'
            ),
            models.CheckConstraint(
                check=~(models.Q(paper_trading_enabled=True) & models.Q(live_trading_enabled=True)),
                name='paper_vs_live_check'
            ),
        ]
```

**Behavior:**  
- If both `auto_purchase_enabled` and `notification_only_enabled` are `True`, an `IntegrityError` is raised.  
- Similarly, enabling both `paper_trading` and `live_trading` simultaneously triggers an error.

---

## **3. Enforcing Logic with `save()` Overrides**

Application-level validations can be enforced by overriding the `save()` method to provide immediate feedback.

### **Example: Overriding `save()` for `TradingPreferences`**

```python
class TradingPreferences(models.Model):
    user = models.OneToOneField(
        User, on_delete=models.CASCADE, primary_key=True, related_name='trading_preferences'
    )
    auto_purchase_enabled = models.BooleanField(default=False)
    notification_only_enabled = models.BooleanField(default=True)
    paper_trading_enabled = models.BooleanField(default=True)
    live_trading_enabled = models.BooleanField(default=False)

    def save(self, *args, **kwargs):
        if self.auto_purchase_enabled and self.notification_only_enabled:
            raise ValueError("Both auto-purchase and notification-only cannot be enabled at the same time.")
        if self.paper_trading_enabled and self.live_trading_enabled:
            raise ValueError("Both paper trading and live trading cannot be enabled at the same time.")
        super().save(*args, **kwargs)
```

**Comparison of Approaches:**  
- **`CheckConstraint`**: Enforces integrity at the database level, protecting against bypassed application logic.  
- **`save()` Override**: Provides user-friendly feedback but requires all operations to go through the application code.

**Recommendation:**  
- Use **both** `CheckConstraint` and `save()` overrides for robust validation.

---

## **4. Summary of Constraints and Handling Techniques**

This section provides a summary of constraints implemented across models and recommended handling strategies.

| **Model**             | **Constraint Type**               | **Constraint Description**                                          | **Handling Strategy**                                  |
|-----------------------|------------------------------------|----------------------------------------------------------------------|-------------------------------------------------------|
| **UserAlgorithmSetting** | Unique Constraint (`unique_together`) | Ensures unique `(user, algorithm)` combinations.                      | Use `update_or_create()` or handle `IntegrityError`. |
| **TradingPreferences** | Check Constraint                  | Prevents both `auto_purchase` and `notification_only` from being enabled. | Catch `IntegrityError` or use `save()` override.      |
|                       |                                    | Prevents both `paper_trading` and `live_trading` from being enabled.  |                                                       |
| **Order**             | Check Constraint                  | Restricts `order_type` to `'buy'` or `'sell'`.                        | Handle `IntegrityError` for invalid values.           |

---

## **5. Practical Example: Handling Integrity Errors**

Below is a general example of handling `IntegrityError` during record creation:

```python
from django.db import IntegrityError

try:
    TradingPreferences.objects.create(
        user=user,
        auto_purchase_enabled=True,
        notification_only_enabled=True  # This will raise IntegrityError
    )
except IntegrityError as e:
    print(f"IntegrityError: {e}")
```

**Outcome:**  
- An `IntegrityError` is raised when the constraints are violated, ensuring data integrity.

---

## **6. Final Recommendations**

- **Unique Constraints**: Use `unique_together` to prevent duplicate records. Handle potential conflicts using `update_or_create()` or by catching `IntegrityError`.
- **Check Constraints**: Apply business rules with `CheckConstraint` to ensure database-level integrity. Use application-level `save()` overrides for user-friendly feedback.
- **Error Handling**: Always handle `IntegrityError` gracefully in production to maintain a seamless user experience.
