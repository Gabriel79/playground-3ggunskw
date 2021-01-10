# Sauvegarde de la dernière page lue

Maintenant que nous pouvons lire un livre sur notre calculatrice, il serait bien de pouvoir sauvegarder la dernière page lue, pour ne pas re-ouvrir le livre à chaque fois au début.

Pour cela nous allons utiliser le système de sauvegarde de la Numworks, le même qui est utilisé par vos scripts pythons. Pour chaque livre, nous allons avoir un "fichier" contenant juste l'offset dans le fichier du dernier emplacement lu.

Vous allez peut être penser, un fichier pour stocker 1 nombre? Oui, ca semble un peu "too much", sur un ordinateur on aurait probablement utiliser un seul fichier pour sauvegarder toutes les offsets sous la forme d'une suite de clé/valeur ou la clé aurait été le nom du fichier texte et la valeur l'offset dans le fichier. En réalité, il n'y a pas vraiment de "fichier" sur la Numworks, juste une zone de mémoire où on peut écrire. L'écriture se fait au moyen de `Record` (enregistrement). Un `Record` a un nom, et une zone de mémoire dont on définit la taille quand on le crée. En pratique, la mémoire contient d'abord le nom du fichier, suivi de la zone de mémoire qui contiendra l'offset. Il n'y pas de perte de place (contrairement à un fichier sur un ordinateur qui a une taille minimale même s'il ne contient pas grand chose).

## Le header

Nous allons créer des `Record` avec pour nom le nom du fichier texte ouvert, mais pour cela nous devons donc garder le nom du fichier sous la main. Enrichissons donc notre `ReadBookController` d'un pointeur vers le fichier ouvert :
```c++
const External::Archive::File* m_file;
```
Et rajoutons 2 méthodes, une pour sauver la position dans le fichier, l'autre pour la charger :
```c++
    void savePosition() const;
    void loadPosition();
```
Remarquez que la méthode qui sauve la position n'est pas supposée modifier le contenu de la classe, on la déclare donc `const`.

Notre classe doit donc ressembler à ça :
```c++
class ReadBookController : public ViewController {
public:
  ReadBookController(Responder * parentResponder);
  View * view() override;

  void setBook(const External::Archive::File& file);
  bool handleEvent(Ion::Events::Event event) override;

  void savePosition() const;
  void loadPosition();
private:
  WordWrapTextView m_readerView;
  const External::Archive::File* m_file;
};
```

## L'implémentation

Dans un premier temps nous allons sauvegarder le fichier dans notre nouvelle variable. Modifions donc la fonction :
```c++
void ReadBookController::setBook(const External::Archive::File& file)
{
    m_file = &file;
    m_readerView.setText(reinterpret_cast<const char*>(file.data), file.dataLength);
}
```

Remarquez que `m_file` est un pointeur, on met donc dedans l'adresse du fichier passé en paramètre, qu'on obtient en utilisant le `&` devant `file` passé en paramètre. 

### Sauvegarde

Ensuite, nous allons écrire la fonction de sauvegarde dans le contrôleur :
```c++
void ReadBookController::savePosition() const
{
```
On commence par essayer de créer un nouveau `Record` pour notre fichier avec le numéro de la page comme donnée. La méthode `createRecordWithFullName` peut enregistrer n'importe quoi dans le `Record`, elle prend donc en paramètre l'adresse de la zone de mémoire à sauvegarder et la taille de cette zone de mémoire. On passe donc en paramètre l'adresse de notre variable `pageOffset` et la taille de cette variable (4 octets) renvoyée par `sizeof(pageOffset)`. Notez que la mémoire sera recopiée dans le `Record`, il n'y donc pas d'inquiétude à avoir par rapport au fait que la variable `pageOffset` est locale et va donc être invalide au sortir de la fonction.

```c++
  int pageOffset = m_readerView.getPageOffset(); 
  Ion::Storage::Record::ErrorStatus status = Ion::Storage::sharedStorage()->createRecordWithFullName(m_file->name, &pageOffset, sizeof(pageOffset));
```
La méthode `getPageOffset` n'existe pas encore, nous la créerons ensuite.

La méthode `createRecordWithFullName` renvoie un status qui nous indiquera si la sauvegarde a été faite, ou si un `Record` de même nom existe déjà. Au quel cas, on va juste le mettre à jour. Pour mettre à jour le record, on crée une structure `Ion::Storage::Record::Data` qui contiendra l'adresse de la mémoire à copier dans le `Record` et sa taille :
```c++
  if(Ion::Storage::Record::ErrorStatus::NameTaken == status)
  {
      Ion::Storage::Record::Data data;
      data.buffer = &pageOffset;
      data.size = sizeof(pageOffset);
      status = Ion::Storage::sharedStorage()->createRecordWithFullName(m_file.name).setValue(data);
  }
}
```

### Chargement

Et voilà! Enfin, il nous faut maintenant lire ce numéro de page pour pouvoir y aller à l'ouverture du livre. Rajoutons donc une méthode : 
```c++
void ReadBookController::loadPosition()
{
```

On récupère ensuite le `Record` correspondant à notre fichier :
```c++
Ion::Storage::Record r = Ion::Storage::sharedStorage()->recordNamed(m_file->name);
```

L'API n'est ensuite pas très explicite. On doit regarder qu'il y a bien un `Record` dans notre `Record`...
```c++:
  if(Ion::Storage::sharedStorage()->hasRecord(r))
  {
```
Si c'est bien le cas, on récupère le contenu du `Record`. La ligne est un peu compliquée. Le contenu du `Record`se trouve dans `r.value().buffer`, on va d'abord dire qu'il faut interpréter cette variable comme un pointeur vers un `const int`(`const` car on ne souhaite pas modifier la valeur), puis on déréférence ce pointeur (`*`):
```c++
      m_pageOffset = *(static_cast<const int*>(r.value().buffer);
  }
```
Si le `Record` n'existe pas encore, on remet l'offset à 0.
```c++
  else
  {
    m_readerView.setPageOffset(0);
  }
}
```

### En résumé

Pour plus de lisibilité, nos 2 fonctions :
```c++
void ReadBookController::savePosition() const
{
  int pageOffset = m_readerView.getPageOffset(); 
  Ion::Storage::Record::ErrorStatus status = Ion::Storage::sharedStorage()->createRecordWithFullName(m_file->name, &pageOffset, sizeof(pageOffset));
  if(Ion::Storage::Record::ErrorStatus::NameTaken == status)
  {
      Ion::Storage::Record::Data data;
      data.buffer = &pageOffset;
      data.size = sizeof(pageOffset);
      status = Ion::Storage::sharedStorage()->recordNamed(m_file->name).setValue(data);
  }
}

void ReadBookController::loadPosition()
{
  Ion::Storage::Record r = Ion::Storage::sharedStorage()->recordNamed(m_file->name);
  if(Ion::Storage::sharedStorage()->hasRecord(r))
  {
    int pageOffset = *(static_cast<const int*>(r.value().buffer));
    m_readerView.setPageOffset(pageOffset);
  }
  else
  {
    m_readerView.setPageOffset(0);
  }
}
```

### Intégration

Il nous reste à les appeler. Chargeons la dernière page lue, après avoir passé le fichier au contrôleur :
```c++
void ReadBookController::setBook(const External::Archive::File& file)
{
    m_file = &file;
    loadPosition();
    m_readerView.setText(reinterpret_cast<const char*>(file.data), file.dataLength);
}
```

Pour la sauvegarde, on va réagir au bouton `back` qui permet de sortir du livre pour revenir à la liste des livres. On va réagir également au bouton `Home` qui permet de sortir de l'application :
```c++
bool ReadBookController::handleEvent(Ion::Events::Event event) 
{  
    if(event == Ion::Events::Down)
    {
        m_readerView.nextPage();
        return true;
    }
    if(event == Ion::Events::Up)
    {
        m_readerView.previousPage();
        return true;
    }
    if(event == Ion::Events::Back || event == Ion::Events::Home)
    {
      savePosition();
    }
    return false;
}
```

## Ajout à la WordWrapTextView

Il nous manque 2 fonctions que nous avons appelé comme si elles existaient... rajoutons les à la `WordWrapTextView` :
```c++
    int getPageOffset() const;
    void setPageOffset(int o);
```

et leur implémentation :
```c++
int WordWrapTextView::getPageOffset() const
{
    return m_pageOffset;
}

void WordWrapTextView::setPageOffset(int o)
{
    m_pageOffset = o;
}
```

Un essai ?

# En peu de ménage

Après des tests ou après avoir lu plein de livre, la mémoire de la calculatrice sera pleine de `Record` ne correspondant plus à des livres actuellement présents sur la calculatrice. On va donc rajouter une fonction pour faire le ménage. Nous aurons également besoin d'une petite fonction accessoire pour nous dire si un fichier est encore présent sur la calculatrice. Commençons par celle-ci que nous rajoutons au `ListBookController` :
```c++
    bool hasBook(const char* filename) const;
```

Puis dans le fichier d'implémentation, nous faisons tout simplement un boucle sur les fichiers contenus le `ListBookController` pour déterminer si le fichier dont le nom `filename` passé en paramètre est présent.
Une remarque sur la fonction `strcmp`, elle renvoie 0 si les deux chaînes sont identiques (ce qui peut être contre-intuitif).

```c++
bool ListBookController::hasBook(const char* filename) const
{
    for(int i=0;i<m_nbFiles;i++)
    {
        if(strcmp(m_files[i].name, filename) == 0)
        {
            return true;
        }
    }
    return false;
}
```

Rajoutons ensuite notre fonction
```c++
  void cleanRemovedBookRecord();
```
dans le header et dans l'implémentation. Sa logique est la suivante, on demande à la Numworks tous les `Record` avec une extension en ".txt" correspondant à nos fichiers texte, pour chacun on va s'assurer qu'ils sont encore présent en mémoire, si ce n'est pas le cas on va supprimer le `Record`.

```c++
void ListBookController::cleanRemovedBookRecord()
{
  int nb = Ion::Storage::sharedStorage()->numberOfRecordsWithExtension("txt");
  for(int i=0; i<nb; i++)
  {
    Ion::Storage::Record r = Ion::Storage::sharedStorage()->recordWithExtensionAtIndex("txt", i);
    if(!hasBook(r.fullName()))
    {
        r.destroy();
    }
  }
}
```

Il n'est pas simple de vérifier le bon fonctionnement de cette fonction. Le site de Numworks permet de voir le contenu de la calculatrice, il doit vous permettre de vous assurer de la bonne suppression de vos vieux `Record` correspondant à d'ancien fichier ".txt" absent de la mémoire.

# Le mot de la fin

Voilà, ce tutoriel est terminé. Cette application pourraient encore bien sûr être améliorée, mais elle permet déjà la lecture de livres au format txt... de quoi vous occuper si vous vous ennuyez en cours... euh entre 2 cours.

Normalement, vous en savez maintenant assez pour réaliser vos propres applications.

Si quelqu'un a été jusque là, je serais content de le savoir. Et si vous avez rencontré des difficultés n'hésitez pas à les signaler.

S'il y a des bugs (fort probable) dites le moi, je chercherais à les corriger !

Sinon voici un lien vers le code source (il y a tout le code d'Oméga, auquel j'ai rajouté mon application) [https://github.com/Gabriel79/OmegaWithReaderTutorial](https://github.com/Gabriel79/OmegaWithReaderTutorial)

Merci
