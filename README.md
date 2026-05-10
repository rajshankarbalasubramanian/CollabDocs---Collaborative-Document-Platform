# CollabDocs---Collaborative-Document-Platform
Builded the backend API for CollabDocs


Project Overview
----------------
CollabDocs is a backend API built using Django REST Framework for a collaborative documentation platform where users can:
- create workspaces,
- add members with roles,
- create and update versioned documents,
- add comments and replies,
- tag documents,
- track actions with audit logs,
- and view request logging middleware output.

Note:
This local implementation uses SQLite because PostgreSQL/admin access was restricted on the office laptop. The design can later be switched to PostgreSQL if needed.

1) Tech Stack
-------------
- Python
- Django
- Django REST Framework
- SQLite (local development)
- Postman for API testing
- VS Code for development

2) Project Structure
--------------------
collabdocs-api/
|
|-- collabdocs/
|   |-- migrations/
|   |-- __init__.py
|   |-- admin.py
|   |-- apps.py
|   |-- middleware.py
|   |-- models.py
|   |-- serializers.py
|   |-- signals.py
|   |-- tests.py
|   `-- views.py
|
|-- config/
|   |-- __init__.py
|   |-- asgi.py
|   |-- settings.py
|   |-- urls.py
|   `-- wsgi.py
|
|-- db.sqlite3
|-- manage.py
`-- requirements.txt

3) Features Implemented
-----------------------
- Custom UUID-based User model
- Workspace and WorkspaceMember with role-based membership
- Document create/update with DocumentVersion auto-creation inside transactions
- Tags (ManyToMany with Document)
- Threaded comments using self-referential parent field
- Audit logs written via signal and explicit actions
- Custom request logging middleware
- Query optimizations with select_related, prefetch_related, Q, Count
- Pagination for list endpoints
- Custom serializer validation and meaningful error messages

4) Prerequisites
----------------
- Python 3.10+ (or compatible version available on your laptop)
- VS Code
- Postman

5) Setup Steps in VS Code
-------------------------
Step 1: Open only the collabdocs-api folder in VS Code.

Step 2: Create/activate a virtual environment
Windows PowerShell:
python -m venv .venv
.\.venv\Scripts\Activate.ps1

Step 3: Install dependencies
pip install django djangorestframework
pip freeze > requirements.txt

Step 4: Ensure settings.py uses SQLite
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
AUTH_USER_MODEL = 'collabdocs.User'

Step 5: Clean reset if needed (only for a new/local project)
- delete db.sqlite3
- delete all files in collabdocs/migrations/ except __init__.py

Step 6: Run migrations
python manage.py makemigrations
python manage.py migrate

Step 7: Create superuser (optional but recommended)
python manage.py createsuperuser

Step 8: Run server
python manage.py runserver

Server URL:
http://127.0.0.1:8000/
Admin URL:
http://127.0.0.1:8000/admin/

6) Core Files Used
------------------
config/settings.py
- SQLite database
- rest_framework in INSTALLED_APPS
- custom middleware registration
- AUTH_USER_MODEL = 'collabdocs.User'

collabdocs/models.py
Contains:
- User
- Workspace
- WorkspaceMember
- Document
- DocumentVersion
- Comment
- Tag
- AuditLog

collabdocs/serializers.py
Contains serializers for all major models, plus validation.

collabdocs/views.py
Contains:
- UserViewSet
- WorkspaceViewSet
- DocumentViewSet
- CommentViewSet
- TagViewSet
- AuditLogViewSet

collabdocs/signals.py
Creates AuditLog automatically after document create/update.

collabdocs/middleware.py
Prints:
- HTTP method
- endpoint path
- response status code
- request time in milliseconds

config/urls.py
Registers all routes using DRF router.

7) Important Troubleshooting Notes from Development
---------------------------------------------------
A. Wrong project folder issue  
Always run commands from inside:  
...\collabdocs-api  

B. If Django still shows PostgreSQL instead of SQLite  
Run:  
python manage.py shell  
from django.conf import settings  
print(settings.DATABASES)  
Expected engine: 'django.db.backends.sqlite3'  

C. Trailing slash matters in DRF routes  
Use: /api/comments/  
not: /api/comments  

D. Custom action route for tags  
Use: POST /api/documents/<document_id>/tags/  

E. Workspace member route must use actual workspace UUID  
Correct: /api/workspaces/<actual-uuid>/members/  

8) API Endpoints Implemented
----------------------------
Users
- POST /api/users/
- GET /api/users/{id}/

Workspaces
- POST /api/workspaces/
- GET /api/workspaces/{id}/
- POST /api/workspaces/{id}/members/
- GET /api/workspaces/{id}/members/
- GET /api/workspaces/{id}/summary/

Documents
- POST /api/documents/
- PUT /api/documents/{id}/
- GET /api/documents/
- GET /api/documents/{id}/versions/
- GET /api/documents/{id}/stats/
- POST /api/documents/{id}/tags/

Comments
- POST /api/comments/
- GET /api/comments/?document={id}

Tags & Audit Logs
- POST /api/tags/
- GET /api/audit-logs/

9) Sample Local IDs Used During Testing
---------------------------------------
- Workspace ID: 2b70f707-5255-48f4-9b01-c697b67e8104
- Owner (raj): 72eb7b26-235c-4ecd-bd7f-787a436ba8ed
- Member (kumar): 04b1bb2a-5946-49ba-b0a2-489df23052cd
- Document ID: cc96c94a-6ea0-4e20-b225-df0932b9ac92
- Tag python: 59452388-ca95-4010-a40c-ada02f85599f
- Tag django: e3947567-7c14-4479-885d-da70f3dc72a2

10) Postman Execution Order (Recommended)
-----------------------------------------
10.1 Create User 1
POST /api/users/
{ "username": "raj", "email": "raj@example.com", "password": "Test@1234" }

10.2 Create User 2
POST /api/users/
{ "username": "kumar", "email": "kumar@example.com", "password": "Test@1234" }

10.3 Create Workspace
POST /api/workspaces/
{ "name": "Engineering Workspace", "owner": "PUT-OWNER-USER-UUID-HERE" }

10.4 Get Workspace
GET /api/workspaces/<workspace_uuid>/

10.5 Add Member to Workspace
POST /api/workspaces/<workspace_uuid>/members/
{ "user": "PUT-USER2-UUID-HERE" }

10.6 List Workspace Members
GET /api/workspaces/<workspace_uuid>/members/

10.7 Workspace Summary
GET /api/workspaces/<workspace_uuid>/summary/

10.8 Create Tag 1
POST /api/tags/
{ "name": "python" }

10.9 Create Tag 2
POST /api/tags/
{ "name": "django" }

10.10 Create Document
POST /api/documents/
{ "workspace": "PUT-WORKSPACE-UUID-HERE", "title": "Project Notes", "content": "Initial content for CollabDocs project.", "status": "draft", "created_by": "PUT-OWNER-USER-UUID-HERE" }

10.11 Update Document
PUT /api/documents/<document_uuid>/
{ "workspace": "PUT-WORKSPACE-UUID-HERE", "title": "Project Notes Updated", "content": "Updated content with more details.", "status": "published", "created_by": "PUT-OWNER-USER-UUID-HERE" }

10.12 Get Versions
GET /api/documents/<document_uuid>/versions/

10.13 Get Stats
GET /api/documents/<document_uuid>/stats/

10.14 Add Tags to Document
POST /api/documents/<document_uuid>/tags/
{ "tag_ids": ["PUT-TAG1-UUID-HERE", "PUT-TAG2-UUID-HERE"] }

10.15 Filter Documents by Workspace
GET /api/documents/?workspace=<workspace_uuid>

10.16 Filter Documents by Status
GET /api/documents/?status=published

10.17 Filter Documents by Tag
GET /api/documents/?tag=python

10.18 Filter Documents by Search
GET /api/documents/?search=project

10.19 Add Top-Level Comment
POST /api/comments/
{ "document": "PUT-DOCUMENT-UUID-HERE", "author": "PUT-USER2-UUID-HERE", "content": "This is a top-level comment." }

