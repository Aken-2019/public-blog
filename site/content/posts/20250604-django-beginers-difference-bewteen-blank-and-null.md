---
title: "Django Beginers - Difference Between Blank and Null"
date: 2025-06-04T14:22:07+08:00
draft: false
---

    中文标题: Blank 和 Null 的区别

Forshort, `null` is for database, `blank` is for form.

Below are common combinations:


**Required field (default)**
```python
name = models.CharField(max_length=100)  # blank=False, null=False
```

**Optional field (with null in database)**
```python
middle_name = models.CharField(max_length=100, blank=True, null=True)
```

**Optional Field with Empty String**
```python
nickname = models.CharField(max_length=100, blank=True, default='')
```

### Important Notes

For string-based fields (CharField, TextField), it's often better to use
`blank=True` with `default=''` instead of `null=True` to avoid having both
`NULL` and empty strings in the database.

For non-string fields (IntegerField, DateField, etc.), you typically use
both `null=True` and `blank=True` to make them optional.

BooleanField typically uses `null=True, blank=True` to allow for three
states: `True`, `False`, and `NULL`/`None`.


### Real Dev Cases

1. For fields which is calculated / queried from other fields, what's the Recommendation?
```python
class YourModel(models.Model):
    # Other fields that the derived field depends on
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    
    # Derived field
    full_name = models.CharField(
        max_length=201,  # First + last + space
        blank=True,      # Allow empty in forms/admin
        editable=False,  # Prevent manual editing
        help_text="Automatically generated from first and last name"
    )
    
    def save(self, *args, **kwargs):
        # Calculate the derived field before saving
        self.full_name = f"{self.first_name} {self.last_name}".strip()
        super().save(*args, **kwargs)
```


Why This Works Best:

**blank=True**:

* Prevents form validation errors since the field isn't user-editable
* The field will be populated programmatically in save()

**editable=False** (Optional but recommended):

* Removes the field from admin/forms since it's auto-generated
* Makes it clear this field shouldn't be manually edited

**null parameter**:

* Omitted (defaults to False) to enforce NOT NULL at the database level
* The field will never be NULL because we set it in save()

### Alternative: Using `@property`

If the field is purely derived and doesn't need to be stored in the database:

```python
class YourModel(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    
    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}".strip()
    
    # No need for save() override or database field
```


For a field that needs to be queried from an external source, here's a recommended approach using Django's `@cached_property` and `@property` decorators:

```python
from django.db import models
from django.utils.functional import cached_property
from django.core.cache import cache
import requests
import logging

class YourModel(models.Model):
    # Other fields that might be used as parameters for the external query
    external_id = models.CharField(max_length=100)
    last_fetched = models.DateTimeField(null=True, blank=True)
    
    # Other model fields...
    
    @cached_property
    def external_data(self):
        """
        Fetches data from external source if not in cache.
        Returns None if the request fails.
        """
        cache_key = f'external_data_{self.external_id}'
        data = cache.get(cache_key)
        
        if data is None:
            try:
                # Replace with your actual external API call
                response = requests.get(
                    f'https://api.example.com/data/{self.external_id}',
                    timeout=5
                )
                response.raise_for_status()
                data = response.json()
                
                # Cache the result (e.g., for 1 hour)
                cache.set(cache_key, data, timeout=3600)
                self.last_fetched = timezone.now()
                self.save(update_fields=['last_fetched'])
                
            except (requests.RequestException, ValueError) as e:
                logging.error(f"Failed to fetch external data: {e}")
                return None
                
        return data

    def refresh_external_data(self):
        """
        Force refresh the external data, bypassing cache.
        """
        cache_key = f'external_data_{self.external_id}'
        cache.delete(cache_key)
        return self.external_data  # This will trigger a fresh fetch

```

