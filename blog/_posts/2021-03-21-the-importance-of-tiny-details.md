---
layout: post
title: The importance of tiny details
tags: jobsearch python pep8 formatting
author: carlos
---

I recently applied for a job at a company in Spain whose name I will not mention. After making the HR cut, they assigned to me some homework that I was supposed to complete in one week, give or take. The requirements for the job were the following: 

```
You must have
· 4+ years of industry experience building data-intensive web applications.
· Proficiency in Python.
· Experience using Python web app frameworks
· Hands-on experience with relational DBs.
· A strong belief that you get the most out of your time when working in a team.
· The ability to work comfortably in Bash and on remote servers.
· Lots of fun explaining our back-end to your grandparents.
· Experience with Linux.


It would be great if you also have
· Basic knowledge of Docker.
· Frequently worked/developed/deployed on the cloud and in automated environments.
· Experience working with continuous integration.
· Experience with software design lifecycle and best practices (code reviews, testing).
· Experience designing and developing RESTful APIs.
talk about using linter tools when submitting code challenges to companies
```

Although it didn't mention it explicitly, after receiving the assignment I found out that what they meant with "Experience using Python web app frameworks" was Django (why not saying it, then?) It's not a big surprise, but since Django is not the only Python web app framework around and it's pretty clear by looking at my resume that I have never worked with it, it took me a bit off guard. In any case, even though I am (or was) a complete Django newbie, I decided to give it a go. I just thought I would learn something from it, and also that the guys at the company were pretty nice giving me the chance, which doesn’t happen every day. 

The challenge was nothing out of the extraordinary, and it was at reach of anybody with some backend experience. It consisted on building some models based on data they wanted to export from files to a database, and then implement an API that they defined using the openAPI standard. Everything pretty clear and unambiguous. 
Working on it hasn't been easy: this got us travelling and once back at home, we had to go into quarantine for 10 days, which seems an ideal situation when you have to work on something, except for the fact that quarantining with a 3.5 year old son and 1 year old daughter makes things slightly more complicated. 

Anyways, after working for 10 days on it, scraping some time here and there, I submitted my assignment. A few days later, I have found out that they are (very politely) giving me their finger. Which typically pisses me off, but I have to admit they have been very cool about it. Moreover, they took the time to explain me what they liked and didn’t like about my solution. 
The likes: 

+ You queried an API to get the conversion rate
+ You added tests to all the code 
+ The API was working (duh)


The didn’t likes:
- Codestyle 
- The code could be organized in a clearer way. Maybe using MVC instead of adding all the logic in views. 
- There are very long methods in views.py

OK, I agree with the second didn’t like, which I would say it is the most important one, and I knew I should have structured the code better. About the long methods, how long is long? I went back to the file they mentioned, and the longest method doesn’t exceed 100 lines, new lines included. Is that that long?
But I honestly didn’t expect the coding style. I didn’t use a formatting tool this time, but since I am familiar with PEP8, it didn’t cross my mind style would be a problem. 
Now, I know that I don’t follow PEP8 100%, and I do things like this: 

```python
from rest_framework             import viewsets

from django.core                import serializers as core_serializers
from django.db                  import IntegrityError
from django.db.models           import Case, DecimalField, F, Sum, When
from django.http.response       import HttpResponse, HttpResponseBadRequest
from django.views.generic       import TemplateView
from django.views.generic.edit  import FormView
from django.shortcuts           import render

from .                          import models, serializers, utils
from .forms                     import SelectDsrsFileForm
```

See how everything is aligned around the “import” keyword? To me, this not only looks nicer, but improves readability. The same happens if some variables are defined one after the other, like:  

```python
form_class  = self.get_form_class()
form        = self.get_form(form_class)
files       = request.FILES.getlist(‘dsr_files’)
```

Just out of curiosity, I used VS Editor and autopep8 on this file, and after the formatter did its job, the previous lines looked like this: 

```python
from rest_framework import viewsets

from django.core import serializers as core_serializers
from django.db import IntegrityError
from django.db.models import Case, DecimalField, F, Sum, When
from django.http.response import HttpResponse, HttpResponseBadRequest
from django.views.generic import TemplateView
from django.views.generic.edit import FormView
from django.shortcuts import render

from . import models, serializers, utils
from .forms import SelectDsrsFileForm


form_class = SelectDsrsFileForm
template_name = 'dsrs/upload-dsrs.html'
success_url = 'success/'
```

As one could expect, nothing dramatically different from one example to the next one. But even they claimed “We'll never reject a candidate for this, but these tiny details help us to see how we worked with Python before.”, it seems a bit suspicious that they mentioned it at all. In any case, kudos to the company for replying to my questions and explaining why they decided to not move forward: in my experience, most people won't bother on explaining anything, and these guys prove not to be hearless robots but human beings, even if they gave me the finger. 

Conclusion: don’t think and use the tools, they are there for a reason. That, and that my aesthetics sense suck and should never he trusted. 

Format wisely!



