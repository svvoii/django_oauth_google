# django_oauth_google
How to setup 3rd party authentication to sign in to the django app with google account.  

In this tutorial I use `django-allauth` to setup google authentication.  


## 0. Prerequisites
- Python 3.8
- Pipenv
- Django 3.1.7
- django-allauth

*Creating new directory for the project and virtual environment*

```bash
mkdir django_oauth_google
cd django_oauth_google
pipenv shell
pipenv install django
pipenv install django-allauth
```

*Some additional packages might be needed for OAuth2 authentication to work:*  

```bash
pipenv install requests
pipenv install PyJWT
pipenv install cryptography
```

## 1. Create a new Django project
```bash
django-admin startproject my_project
cd my_project
```


## 2. Creating a new app

```bash
python manage.py startapp user_app
```

*This app will render the profile page after the user is authenticated.*  
*We will add `profile.html` in the `user_app/templates/user_app` directory.*     

*In your project this might be the app where the user management is implemented.*  


## 3. Adding the following to the `my_project/settings.py`

```python
# in the INSTALLED_APPS
INSTALLED_APPS = [
	...
	# django-allauth apps
	'allauth',
	'allauth.account',
	'allauth.socialaccount',
	'allauth.socialaccount.providers.google', # this is for google authentication
	
	# our apps
	'user_app',
]

MIDDLEWARE = [
	...
	# django-allauth middleware
	'allauth.account.middleware.AccountMiddleware',
]

...
# other allauth settings

AUTHENTICATION_BACKENDS = [
	'django.contrib.auth.backends.ModelBackend', # Needed to login by username in Django admin, regardless of `allauth`
	'allauth.account.auth_backends.AuthenticationBackend', # `allauth` specific authentication methods, such as login by e-mail
]

SITE_ID = 1 # this is the default site id (must be set if we use `django.contrib.sites` in INSTALLED_APPS above !!)

ACCOUNT_AUTHENTICATION_METHOD = 'email' # this will allow the user to login with email
ACCOUNT_EMAIL_REQUIRED = True # this will require the user to provide email

# this is for testing purposes, the email which suppose to be sent to the user will be printed in the console
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
# The link from the console can then be copied and pasted in the browser to verify the email
```


## 4. Adding the following to the `my_project/urls.py`

*The `my_project/urls.py` will look like this:*

```python
from django.contrib import admin
from django.urls import path, include
from user_app import views


urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('allauth.urls')), # this will include all the urls provided by django-allauth
	path('accounts/profile/', views.profile, name='profile'), # this is the profile page after the user is authenticated
]
```


## 5. Adding the following to the `user_app/views.py`

*This is how the `user_app/views.py` will look like:*  

```python
from django.shortcuts import render

def profile(request):
	return render(request, 'user_app/profile.html')
```

*This will simply render the `profile.html` page.*  


## 6. Adding the following to the `user_app/templates/user_app/profile.html`

*Create a new directory `user_app/templates/user_app` and add `profile.html` in it.*  

```html
<h1>Logged in as: {{ request.user.username }}</h1>

<a href="{% url 'account_logout' %}">Logout</a>
```

*This will display the username of the user and a logout link.*  


## 7. Migrate the database

```bash
python manage.py migrate
```


## 8. Create a superuser

```bash
python manage.py createsuperuser
```

*This will create a superuser to access the admin page.*  


**NOTE:** *At this point we can run the server and access the admin page at `127.0.0.1:8000/admin`*  
*On the admin page we will beable to add new social application in the `Social applications` section. For that we first need to obtain the `client_id` and `client_secret` from the google developer console.*  


## 9. Adding Social Application in django admin panel:

- **Go to localhost:8000/admin/ and login with the superuser credentials.**  
*Then go to --> Social applications --> Add social application*  

*Fill in the following fields:*  
`Provider: Google`  
`Name: Google`  
`Client id: 'your_client_id' see below how to get it`   
`Secret key: 'your_secret_key' see below how to get it`  
`Sites: (choose the available `example.com` site on the left side `Available sites` and move it to the right side `Chosen sites` by clicking on the arrow pointing to the right side)` !!  

- **Getting client_id and secret_key from Google Cloud Console:**
*To get the client_id and secret_key go to your -> Google Cloud Console.*  

*Navigate to APIs & Services --> Credentials --> Create credentials --> OAuth client ID*  

**NOTE:** *You might need to create NEW PROJECT if this is your first time and there are no projecte created yet.*  

*With the new project created navigate to APIs & Services and click ENABLE APIS AND SERVICES and search for Google+ API and enable it.*  

*Then go to Credentials and click on CREATE CREDENTIALS and select OAuth client ID.*  
*(If there is no consent screen created yet, create it by clicking on CONFIGURE CONSENT SCREEN and fill in the necessary fields.)*  

*When thet is done select Web application as the application type.*  
*Customize the Name if needed.*  

*Add the following to the Authorized redirect URIs:*  

`http://127.0.0.1:8000/accounts/google/login/callback/`
**NOTE:** *must be 127.0.0.1 and not localhost !!!*  
*For further project deployment this will need to be changed to the actual domain name.*  

*Once you hit Create you will get the client_id and secret_key which you can use to fill in the fields in the django admin panel.*  

*Once the social application is created we can navigate to :*  
`localhost:8000/accounts/login/`  
*..there should be `Google` button in `Or use a third-party` section which will redirect us to the Google login page.*  

*The user will be redirected to the profile page after the login.*  

*DONE!*  

