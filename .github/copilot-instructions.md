# Django Project Management System - AI Agent Instructions

## Project Overview

A Django 5.1 web application for managing software development projects. The system organizes work through **Stages** (etapas) → **Activities** (actividades) → **Tasks** with team collaboration, document management, and meeting scheduling.

**Architecture**: Single app (`myapp`) monolithic structure with PostgreSQL backend + Bootstrap 5 frontend.

## Data Model Architecture

### Core Hierarchy
```
Project (has many)
  ├─ Studentuser (project members)
  ├─ Task (organized by Activity)
  ├─ Document (project deliverables)
  └─ Meeting (team synchronization)

Stage (template - system-wide)
  └─ Activity (reusable project templates)
      ├─ Task (actual project work)
      └─ Multimedia (learning resources)
```

**Key Pattern**: Activities are reusable templates linked to stages; tasks are concrete instances tied to specific projects.

### Critical Field Behaviors
- **Task.status**: Only values are `"Pendiente"`, `"En proceso"`, `"Completado"` (hardcoded in views)
- **Task.date_completed**: Auto-set to `timezone.now()` when status changes to `"Completado"`
- **Project.activo**: Boolean flag (queries may filter active projects)
- **Studentuser**: Replaces previous "Integrantes" model (renamed in migration 0015)

## View Patterns & Workflows

### Request Handling Conventions
- **Form Submission**: Mix of form-based and AJAX-based (`JsonResponse`)
- **Authentication**: All main views require `@login_required` decorator (exceptions: `home`, `registro`, `inicio_sesion`, `about`)
- **User Context**: Most templates receive `data_user` dict from `user_data()` helper (returns `nombre_completo`, `email`)

### AJAX Endpoints Return Pattern
```python
JsonResponse({
    "success": True/False,
    "message": "...",
    "error": "..." (on failure)
})
```
Used by: project/activity/task/resource creation, edits, deletions.

### Common View Sequence
1. **GET**: Render form template with current object/context
2. **POST**: Update object via `Model.objects.create()` or `instance.save()`
3. **Return**: Either `redirect()` or `JsonResponse()`

## Database Configuration

**Development**: PostgreSQL (hardcoded in settings.py)
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'projectdb',
        'USER': 'postgres',
        'PASSWORD': 'P4l4c10s',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

**Production**: Reads `DATABASE_URL` environment variable via `dj_database_url.config()`

## Critical Developer Workflows

### Setup & Initialization
```cmd
virtualenv -p "path/Python312/python.exe" env
env\Scripts\activate
pip install -r requirements.txt
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

### Common Management Commands
- `python manage.py makemigrations` - Detect model changes
- `python manage.py migrate` - Apply migrations (21+ existing)
- `python manage.py createsuperuser` - Admin access
- `python manage.py collectstatic` - Gather static files for production

### Key Files
- `myapp/migrations/0021_meeting_title.py` - Latest migration (meeting title added)
- `mysite/settings.py` - Environment detection: `DEBUG = 'RENDER' not in os.environ`
- `myapp/admin.py` - Register models for Django admin

## Project-Specific Conventions

### URL Routing Patterns
- RESTful-like but inconsistent: `/projects/create/` vs `/activities/create`
- Detail views: `/projects/<id>/info/general`, `/projects/<id>/edit`
- Nested resources: `/projects/<id>/info/general/activity/<id_activity>/tasks`
- JSON endpoints: `/projects/json/`, `/activities/list/json/`, `/activity/<id>/resources/list`

### Task Statistics Queries
Used in `project_info()` - queries tasks by status to calculate progress:
```python
tasks.aggregate(
    absolutely=Count("id"),
    earring=Count("id", filter=Q(status="Pendiente")),
    in_progresss=Count("id", filter=Q(status="En proceso")),
    completed=Count("id", filter=Q(status="Completado"))
)
```
**Bug**: `earring` should be `pending` (typo).

### Frontend Integration
- Bootstrap 5 for styling
- jQuery for AJAX + modals (`data-toggle="modal"`)
- Datatable with action buttons (edit/delete/view dropdowns)
- No React/Vue - vanilla JS mixed with Django templates

## Common Mistakes to Avoid

1. **Hardcoded Status Values**: Always use `"Pendiente"`, `"En proceso"`, `"Completado"` (case-sensitive)
2. **Missing `@login_required`**: Add to all protected views
3. **Migrations**: Run `makemigrations` then `migrate` - never skip migrations
4. **User Context**: Always call `user_data(request)` for templates requiring user info
5. **Typos in Variable Names**: The codebase has naming inconsistencies (`id_*`, `*_id`, `pk`)

## Extension Points

### New Features
- Add models to `myapp/models.py`, create migration, update admin
- Create views with `@login_required`, add URL pattern in `myapp/urls.py`
- Use `JsonResponse` for AJAX, `render()` for forms
- Return values: `status/success` (consistency varies - follow context)

### Database Queries
Use `select_related()` for foreign keys, `prefetch_related()` for reverse relationships (already done in views like `project_info`, `projects_json`)

### Production Deployment
- Set `SECRET_KEY` and `DATABASE_URL` environment variables
- `DEBUG = False` when `RENDER` in environment variables
- Static files served by WhiteNoise (configured in middleware)
