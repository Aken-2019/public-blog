---
title: "Django Beginers - Difference Between login_required and LoginRequiredMixin"
date: 2025-06-04T15:04:17+08:00
draft: false
---

    中文标题: login_required 和 LoginRequiredMixin 的区别


Both login_required and LoginRequiredMixin serve the same purpose of restricting access to authenticated users, but they're used in different contexts:

1. `@login_required` (Function-Based Views)

- Type: Decorator

- Usage: Used with function-based views

- How it works:
    - If user is not logged in, redirects to `settings.LOGIN_URL`
    - Passes the current path in the query string as `?next=/path/`

Example:
```python
from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    return HttpResponse('This view requires login')
```

2. `LoginRequiredMixin` (Class-Based Views)
- Type: Mixin class
- Usage: Used with class-based views
- How it works:
Must be the left-most mixin in the inheritance list
Implements `dispatch()` to check authentication
Also redirects to settings.LOGIN_URL if not authenticated
Example:
```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView

class MyView(LoginRequiredMixin, TemplateView):
    template_name = 'my_template.html'
    login_url = '/login/'  # Optional: override default login URL
    redirect_field_name = 'next'  # Optional: change the "next" parameter name
```
Key Differences:
- Syntax:
    - `@login_required` is a decorator for functions
    - `LoginRequiredMixin` is a class to inherit from
- Flexibility:
    - `LoginRequiredMixin` can be combined with other mixins
    - `@login_required` can't be used with class-based views directly (use @method_decorator instead)
- Customization:
    - Both allow customizing login_url and redirect_field_name
    - `LoginRequiredMixin` makes these class attributes
- Use with other decorators:
    - `@login_required` can be stacked with other decorators
    - `LoginRequiredMixin` can be combined with other mixins
- Best Practices:
    - Use `@login_required` for function-based views
    - Use `LoginRequiredMixin` for class-based views
    - For class-based views that already have a parent class, make sure `LoginRequiredMixin` comes first:
```python
class MyView(LoginRequiredMixin, View):
    # Correct: LoginRequiredMixin first
    pass
```
Common Gotcha:
Don't do this with class-based views:

```python
@method_decorator(login_required, name='dispatch')  # Works but not idiomatic
class MyView(View):
    pass
```
Instead, use:

```python
class MyView(LoginRequiredMixin, View):  # Preferred way
    pass
```