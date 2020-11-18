# Empilons une nouvelle vue

Nous pouvons maintenant choisir un fichier à ouvrir! Une fois ouvert, nous allons avoir besoin d'un nouvel écran, soit une nouvelle vue et un nouveau contrôleur. Commençons par le contrôleur, créons nos 2 fichiers : `apps\reader\read_book_controler.h` et `apps\reader\read_book_controler.cpp`. Rajoutons tout de suite ce nouveau fichier .cpp au Makefile :\
```Makefile
app_sreader_src = $(addprefix apps/reader/,\
  app.cpp \
  list_book_controller.c \
  utility.cpp \
  read_book_controller.c \
)
```

Dans notre nouveau header, rajoutons les includes guards, le namespace et déclarons une nouvelle classe : `ReadBookController` dérivant de `ViewController`:
```c++

```