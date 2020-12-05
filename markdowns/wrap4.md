# Word wrapping Textview - partie 4

Nous pouvons maintenant faire défiler les pages vers l'avant. Il nous faut faire la même chose vers l'arrière. Malheureusement ce n'est pas aussi simple, en effet, vers l'avant c'est le rendu du texte qui nous indique où nous nous sommes arrêtés et où commencera la page suivante. Pour revenir en arrière, nous pourrions nous souvenir d'où commençait la page précédente, mais cela ne marcherait qu'une fois.

Nous allons donc coder un word wrapping à l'envers, c'est à dire qui commence de la fin de la page et remonte jusqu'à sortir de l'écran par le haut! Nous trouverons ainsi le début de la page.

L'algorithme ressemble au précédent, à la différence que nous partons du bas de l'écran à droite et nous calculons les longueurs de mots vers la gauche en remontant quand la ligne est pleine.

Rajoutons donc notre fonction dans le header (`word_wrap_view.h`) :
```c++
   void previousPage();
```

Et son implémentation dans `word_wrap_view.cpp`:
```c++
void WordWrapTextView::previousPage()
{
```
Commençons par quitter la fonction si nous sommes déjà au début du texte.
```c++
    if(m_pageOffset <= 0)
        return;
```
Calculons la largeur d'un espace avec la police (`m_font`) actuelle :
```c++
    const int spaceWidth = m_font->stringSize(" ").width();
```
Puis trouvons la fin du mot de la page en cours, puis son début en utilisant la fonction `UTF8Helper::BeginningOfWord` :
```c++
    const char * endOfWord = text() + m_pageOffset - 1;
    const char * startOfWord = UTF8Helper::BeginningOfWord(text(), endOfWord);
```
Initialisons une variable indiquant on nous nous trouvons dans la page (en bas à droite) en tenant compte de la marge :
```c++
KDPoint textPosition(m_frame.width() - margin, m_frame.height() - margin);
    
```
Puis commençons notre boucle qui s'arrêtera si on remonte au début du texte :
```c++
    while(startOfWord>=text())
    {
```
A partir du début du mot, retrouvons en la fin (normalement ca ne devrait pas servir), calculons la taille de ce mot, et calculons la position à l'écran du début du mot.
```c++
        endOfWord = UTF8Helper::EndOfWord(startOfWord);
        KDSize textSize = m_font->stringSizeUntil(startOfWord, endOfWord);
        KDPoint previousTextPosition = KDPoint(textPosition.x()-textSize.width(), textPosition.y());
```
Si le mot sort de l'écran (par la gauche), on remonte d'une ligne.
```c++
        if(previousTextPosition.x() < margin)
        {
            textPosition = KDPoint(m_frame.width() - margin, textPosition.y() - textSize.height());
            previousTextPosition = KDPoint(textPosition.x() - textSize.width(), textPosition.y());
        }
```
Si en remontant d'une ligne, on sort de l'écran, on sort de la boucle.
```c++
        if(textPosition.y() - textSize.height() < margin)
        {
            break;
        }
```
Lisons les potentiels espaces avant le mot :
```c++
        --startOfWord;
        while(startOfWord >= text() && *startOfWord == ' ')
        {
            previousTextPosition = KDPoint(previousTextPosition.x() - spaceWidth, previousTextPosition.y());
            --startOfWord;
        }
```
Si cela nous amène à sortir de l'écran par la gauche, on remonte d'une ligne.
```c++
        if(previousTextPosition.x() < margin)
        {
            previousTextPosition = KDPoint(m_frame.width() - margin, previousTextPosition.y() - textSize.height());
        }
```
On lit ensuite les sauts de ligne.
```c++
        while(startOfWord >= text() && *startOfWord == '\n')
        {
          previousTextPosition = KDPoint(m_frame.width() - margin, previousTextPosition.y() - textSize.height());
          --startOfWord;
        }
```
Si cela nous conduit à sortir de l'écran par le haut, on sort de la boucle.
```c++
        if(previousTextPosition.y() - textSize.height() < margin)
        {
            break;
        }
```
Pour finir notre boucle, nous mettons à jour notre position, la fin du prochain mot à traiter et cherchons le début de ce mot.
```c++
        textPosition = previousTextPosition;
        endOfWord = startOfWord;
        startOfWord = UTF8Helper::BeginningOfWord(text(), endOfWord);
    }
```
Si nous sommes sortis de la boucle, c'est que le dernier mot traité ne rentrait pas. Le premier mot à afficher est donc celui d'après (dans le sens du texte). On met donc à jour l'offset de cette façon :
```c++

    if(startOfWord == text())
        m_pageOffset = 0;
    else
        m_pageOffset = UTF8Helper::EndOfWord(startOfWord) - text() + 1;
```
Avant d'indiquer qu'il faut redessiner l'écran :
```c++
    markRectAsDirty(bounds());
}
```

Et voilà ! En un seul bloc, cette fonction est ainsi :
```c++
void WordWrapTextView::previousPage()
{
    if(m_pageOffset <= 0)
        return;

    const int spaceWidth = m_font->stringSize(" ").width();

    const char * endOfWord = text() + m_pageOffset;
    const char * startOfWord = UTF8Helper::BeginningOfWord(text(), endOfWord);
    
    KDPoint textPosition(m_frame.width() - margin, m_frame.height() - margin);

    while(startOfWord>=text())
    {
        endOfWord = UTF8Helper::EndOfWord(startOfWord);
        KDSize textSize = m_font->stringSizeUntil(startOfWord, endOfWord);
        KDPoint previousTextPosition = KDPoint(textPosition.x()-textSize.width(), textPosition.y());
        
        if(previousTextPosition.x() < margin)
        {
            std::cout<<"|||\n"<<std::endl;
            textPosition = KDPoint(m_frame.width() - margin, textPosition.y() - textSize.height());
            previousTextPosition = KDPoint(textPosition.x() - textSize.width(), textPosition.y());
        }
        if(textPosition.y() - textSize.height() < margin)
        {
            break;
        }

        
        --startOfWord;
        while(startOfWord >= text() && *startOfWord == ' ')
        {
            previousTextPosition = KDPoint(previousTextPosition.x() - spaceWidth, previousTextPosition.y());
            --startOfWord;
        }
        if(previousTextPosition.x() < margin)
        {
            previousTextPosition = KDPoint(m_frame.width() - margin, previousTextPosition.y() - textSize.height());
        }
        
    
        while(startOfWord >= text() && *startOfWord == '\n')
        {
          previousTextPosition = KDPoint(m_frame.width() - margin, previousTextPosition.y() - textSize.height());
          --startOfWord;
        }    

        if(previousTextPosition.y() - textSize.height() < margin)
        {
            break;
        }

        textPosition = previousTextPosition;
        endOfWord = startOfWord;
        startOfWord = UTF8Helper::BeginningOfWord(text(), endOfWord);
    }
    if(startOfWord == text())
        m_pageOffset = 0;
    else
        m_pageOffset = UTF8Helper::EndOfWord(startOfWord) - text() + 1;
    markRectAsDirty(bounds());
}
```

Il nous reste à appeler notre nouvelle fonction depuis le `ReadBookController`. Rajoutons donc la gestion de l'événement `Ion::Events::Up` :
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
    return false;
}
```

A suivre ...



