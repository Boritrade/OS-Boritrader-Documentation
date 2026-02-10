# **User Account Deletion Documentation**

## **Overview**
This document outlines the process for deleting a user account, including **frontend actions**, **backend processing**, and **handling Stripe subscriptions**. The system ensures **secure account removal** while allowing Stripe data retention for 30 days before permanent deletion.


### **What Happens When an Account is Deleted?**
✅ The user’s account is **permanently removed from the database**.  
✅ Their **Stripe customer profile is marked as deleted** instead of being immediately erased.  
✅ A **scheduled job fully deletes their Stripe profile** after 30 days.  

### **GDPR & Data Protection Considerations**
- **User Consent:** Users must confirm deletion before it is processed.
- **Retention Policy:** Stripe data is retained for **30 days** before permanent removal.

---

## **1. User-Initiated Account Deletion**
### **Frontend Process**
1. The user navigates to the **Settings** page.
2. They click the **"Delete Account"** button.
3. A confirmation popup appears:
   - **Message:** “Are you sure you want to delete your account? This action is irreversible.”
   - **Options:** Confirm or Cancel.
4. If the user confirms:
   - A **POST request** is sent to `/delete-account/` with a CSRF token.
5. If the request is successful:
   - The user is redirected to the **registration page**.
   - The account is permanently deleted.
6. If an error occurs, an alert is displayed.

### **Relevant Code (JavaScript)**
```js
const deleteAccountBtn = document.getElementById('delete-account-btn');

if (deleteAccountBtn) {
    deleteAccountBtn.addEventListener('click', function () {
        if (confirm("Are you sure you want to delete your account? This action is irreversible.")) {
            fetch('/delete-account/', {
                method: "POST",
                headers: {
                    "X-CSRFToken": document.querySelector('[name=csrfmiddlewaretoken]').value,
                    "Content-Type": "application/json"
                }
            })
            .then(response => {
                if (response.ok) {
                    window.location.href = "/register/"; // Redirect to registration page
                } else {
                    alert("Error deleting account. Please try again.");
                }
            })
            .catch(error => alert("Network error: " + error.message));
        }
    });
}
```

---

## **2. Backend Process**
### **Django View Handling Account Deletion**
The `delete_account_view` function in **views_auth.py** processes local account deletion and sends a signal to `signal.py`.

```python
from django.contrib.auth.decorators import login_required
from django.http import JsonResponse
from django.contrib.auth import logout
from django.shortcuts import redirect

@login_required
def delete_account_view(request):
    if request.method == "POST":
        user = request.user

        try:
            user.delete()  # Delete the user account
            logout(request)  # Log out the user
            return JsonResponse({"success": True})  # Success response
        except Exception as e:
            return JsonResponse({"success": False, "error": str(e)}, status=500)
    
    return JsonResponse({"success": False, "error": "Invalid request"}, status=400)
```

### **Django URL Configuration (urls.py)**
```python
path('delete-account/', views_auth.delete_account_view, name='delete_account'),
```

---

## **3. Stripe Subscription Handling** `signals.py`
### **What Happens to Stripe Data?**
✅ The subscription is **set to cancel at the end of the billing cycle** instead of being deleted immediately.  
✅ The **Stripe Customer metadata** is updated to `{ "status": "deleted", "deleted_at": "<timestamp>" }`.  
✅ A **background task will delete Stripe data** after 30 days.  
```python
...
@receiver(pre_delete, sender=User)
def cancel_stripe_subscription(sender, instance, **kwargs):
    """Cancels the user's Stripe subscription when they delete their account."""
    try:
        profile = instance.profile
        if profile.stripe_subscription_id:
            stripe.Subscription.modify(profile.stripe_subscription_id, cancel_at_period_end=True)
            
            # Mark the Stripe customer as deleted
            stripe.Customer.modify(
                profile.stripe_customer_id,
                metadata={"status": "deleted", "deleted_at": str(now())}
            )

            logger.info(f"Marked Stripe customer {profile.stripe_customer_id} as deleted, subscription canceled for user {instance.username}")
    except UserProfile.DoesNotExist:
        logger.info(f"No Stripe profile found for user {instance.username}")
    except stripe.error.StripeError as e:
    logger.error(f"Stripe error while marking user as deleted: {e}")
```

---

## **4. Auto-Deleting Stripe Customers After 30 Days**
A background job **removes old Stripe customers** that were flagged as deleted.

### **Scheduled Cleanup Task (Celery)**
```python
from datetime import timedelta, datetime
from django.utils.timezone import now
import stripe
from django.conf import settings

stripe.api_key = settings.STRIPE_SECRET_KEY

def remove_old_stripe_customers():
    """Deletes Stripe customers marked as 'deleted' after 30 days."""
    customers = stripe.Customer.list(limit=100)
    expiration_date = now() - timedelta(days=30)

    for customer in customers.auto_paging_iter():
        if customer.metadata.get("status") == "deleted":
            deleted_at = customer.metadata.get("deleted_at")
            if deleted_at:
                deleted_at_dt = datetime.strptime(deleted_at, "%Y-%m-%d %H:%M:%S.%f")
                if deleted_at_dt < expiration_date:
                    try:
                        stripe.Customer.delete(customer.id)
                        print(f"Deleted Stripe customer {customer.id}")
                    except stripe.error.StripeError as e:
                        print(f"Error deleting Stripe customer {customer.id}: {e}")
```

### **Scheduling This Task (Celery Beat)**
```python
from celery import shared_task
from .utils import remove_old_stripe_customers

@shared_task
def cleanup_deleted_stripe_customers():
    remove_old_stripe_customers()
```
```python
from celery.schedules import crontab

app.conf.beat_schedule = {
    "clean_old_stripe_customers": {
        "task": "dashboard.tasks.cleanup_deleted_stripe_customers",
        "schedule": crontab(hour=0, minute=0),  # Runs at midnight
    },
}
```

---

## **7. Future Improvements**
✅ **Soft Delete:** Instead of permanent deletion, mark users as "inactive" for a grace period.  
✅ **Email Notification:** Send an email before **final deletion of Stripe data**.  
✅ **User Reactivation:** Allow users to restore their account within 30 days.  