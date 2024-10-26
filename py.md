## Get the repo
```bash
git clone <URL-della-repo>

cd repo

git checkout -b new_branch
```


## Start the py Venv 
```bash
python3 -m venv env
source env/bin/activate
pip install -r requirements.txt
```

# Read from file

## JSON

### Read

```python
import json

# Better for single string in json file
with open('test.json','r') as file:
    obj = json.loads(file.read())
    print(obj)

# Better for complex data in Json file
with open('file.json') as f:
    data = json.load(f)
    print(data)
```

### Write
```python
with open('file.json', 'w') as f:
    json.dump(data, f)
```

## CSV

### Read
```python
import csv
# Better option
with open("address.csv") as csv_file:
    csv_reader_object = csv.DictReader(csv_file, delimiter = ",")
    for row in csv_reader_object:
        print(row)

# Also this works
with open('file.csv') as csvfile:
    reader = csv.reader(csvfile)
    for row in reader:
        print(row) 
```

### Write
```python
with open('file.csv', mode='w', newline='') as csvfile:
    writer = csv.writer(csvfile)
    writer.writerow(['colonna1', 'colonna2'])
```
## Txt

### Read

```python
# Read as 1 string
with open("file_di_testo.txt", "r") as file:
    contenuto = file.read()
    print(contenuto)

# Read line per line 
with open("file_di_testo.txt", "r") as file:
    for linea in file:
        print(linea.strip())  # strip() rimuove eventuali spazi bianchi alla fine

# Create list with lines
with open("file_di_testo.txt", "r") as file:
    righe = file.readlines()  # Restituisce una lista di righe
    print(righe)

```

### Write

```python
# Append contnent to the file
with open("file_di_testo.txt", "a") as file:
    file.write("\nNew line at and of the file.")
```



# TimeStamp and TimeZone
```python
from dateutil import tz
from datetime import datetime

## Time zone locale del computer
local_zone = tz.tzlocal()
local_time = datetime.now(local_zone)
print("Time Zone Locale:", local_time.strftime('%Y-%m-%d %H:%M:%S %Z%z'))

## Time zone di Londra
london_zone = tz.gettz('Europe/London')
london_time = datetime.now(london_zone)
print("Time Zone di Londra:", london_time.strftime('%Y-%m-%d %H:%M:%S %Z%z'))

## Conversione dall'orario locale all'orario di Londra
london_time_converted = local_time.astimezone(london_zone)
print("Orario locale convertito in Londra:", london_time_converted.strftime('%Y-%m-%d %H:%M:%S %Z%z'))
```

# Django 

## Run Django
```bash
python manage.py runserver
```
## Create new App
python manage.py startapp nome_app

settings.py
```python
...
INSTALLED_APPS = [
    # altre app
    'nome_app',
]
...
```

## Create a new page in main project (View and url)
views.py
```python
from django.shortcuts import render

def nuova_view(request):
    return render(request, 'nome_template.html', {})
```
urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('nuova_pagina/', views.nuova_view, name='nuova_pagina')
]
```
nome_template.html in templates/nome_dell_app
```html
<!DOCTYPE html>
<html>
<head><title>Nuova Pagina</title></head>
<body>
    <h1>Benvenuto nella nuova pagina</h1>
</body>
</html>
```

## Create a new page in application (View and url)

nome_app/views.py
```python
from django.shortcuts import render
from django.http import HttpResponse

def my_view(request):
    return HttpResponse("Hello from my new app!")
```

nome_app/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('my-view/', views.my_view, name='my_view')
]
```

urls.py del progetto principale
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('nome_app/', include('nome_app.urls')),
]
```


# Models 
nome_app/models.py
```python
from django.db import models

class ExampleModel(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    is_active = models.BooleanField(default=True)
    price = models.DecimalField(max_digits=5, decimal_places=2)
```

## Apply models change
```python
python manage.py makemigrations
python manage.py migrate
```
# Django Admin

## Create Superuser (admin)
```python
python manage.py createsuperuser
```

## Register new module
nome_app/admin.py
```python
from django.contrib import admin
from .models import ExampleModel

class ExampleModelAdmin(admin.ModelAdmin):
    list_display = ('name', 'created_at', 'is_active')
    search_fields = ('name',)

admin.site.register(ExampleModel, ExampleModelAdmin)
```

# Upload content

Pip module pillow requested! 
settings.py
```python
import os

# Configura la directory per i file media
MEDIA_URL = '/media/'  # URL di base per servire i file media
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')  # Cartella fisica per i file media
```

urly.py progetto 

```python
from django.conf import settings
from django.conf.urls.static import static
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('upload/', include('nome_app.urls')),  # Assumendo che "upload" sia parte del progetto principale
]

# Aggiungi questa riga per servire i file media durante lo sviluppo
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

nome_app/forms.py
```python
from django import forms

class ImageUploadForm(forms.Form):
    image = forms.ImageField()
```

nome_app/views.py
```python
from django.shortcuts import render
from django.http import HttpResponse
from .forms import ImageUploadForm
from django.conf import settings
import os

def upload_view(request):
    if request.method == 'POST':
        form = ImageUploadForm(request.POST, request.FILES)
        if form.is_valid():
            image = form.cleaned_data['image']
            path = os.path.join(settings.MEDIA_ROOT, 'img', image.name)
            
            # Salva l'immagine nella cartella 'media/img'
            with open(path, 'wb+') as destination:
                for chunk in image.chunks():
                    destination.write(chunk)
            
            # Genera il link assoluto
            image_url = request.build_absolute_uri(settings.MEDIA_URL + 'img/' + image.name)
            return HttpResponse(f"L'immagine è stata caricata con successo: <a href='{image_url}'>{image_url}</a>")
    else:
        form = ImageUploadForm()
    return render(request, 'nome_app/upload.html', {'form': form})
```

templates/nome_app/upload.html
```html
<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <title>Carica Immagine</title>
</head>
<body>
    <h1>Carica un'immagine</h1>
    <form method="post" enctype="multipart/form-data">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Carica</button>
    </form>
</body>
</html>
```


nome_app/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.upload_view, name='upload'),
]
```

# Export project

```python
pip freeze > requirements.txt
# Add all to your new commit 
git add .
git commit -m "Descrizione delle modifiche"
# Pusch commit
git push origin nome_del_branch

```
