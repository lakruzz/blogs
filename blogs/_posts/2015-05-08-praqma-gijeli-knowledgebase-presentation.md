---
layout: post
title: Praqma gijeli knowledgebase presentation
tags: [Gijeli, Wiki, MarkDown, Ruby]
author: Bue Petersen
---



## Agenda

**Goals of presentation**:

* learn how edit and add content
* learn how to serve it as a webpage


Work flow of presentation:

* pinside and praqma/gijeli docker container is two independent and different things
    * pinside is a plain git repository with markdown files with our knowlegde (and some other content to make nice webpages)
    * praqma/gijeli is a container we have made that can serve jekyll websites, offer git, ruby and ligquid tool support

* **Edit or add content**
    * open the pinside praqma github private repository
    * browse it, try to edit or add and file
    * clone it
    * edit files with favorite editor
    * push content

* **serve a jekyll website**:
    * using your local installed tools, jekyll ruby and liquid
    * using praqma/gijeli container (asuming you already are able to run docker containers - it is not part of this presentation)


That's it folks!

* Praqma/gijeli container is maintained by Praqma, you can request features and support on support@praqma.net https://github.com/Praqma/docker-gijeli

* Our knowledgebase project in Praqma is maintained in a Trello board, discuss and do feature requests there: https://trello.com/b/ZRHTTLuh/establish-a-praqma-knowledgebase
