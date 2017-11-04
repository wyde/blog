---
title: Note of Writing Your First Django App
tags:
---

Writing your first Django app
===

---
- design
- coding
    - models.py
    - views.py
    - urls.py
    - template
- test
- deploy
- packaging

---

## part 1:
- create project: mysite
- create app: polls
- `polls/view.py`
- `polls/urls.py` (create new file)
    ```
        urlpatterns = [
            url(r'^$', views.index, name='index'),
        ]
    ```
    `name` 接的是 polls/views.py 裡的 funciton name
- `mysite/urls.py`
    - tips: must import url module
- `mysite/settings.py`
    - `ALLOWED_HOSTS = ['140.112.31.217']`
    - `TIME_ZONE = 'Asia/Taipei'`
        - `$ ls /usr/share/zoneinfo` 可以看到合法的時區設置
    - `INSTALLED_APPS = [
        'polls.apps.PollsConfig',
        ...
    ]`

- `./manage.py runserver 0:8000` 開測試 server

---

## part 2:

- polls/models.py
    - `$ ./manage.py makemigrations polls`
        - 如果 model 有變，創建 readable scripts (.py) in polls/migrations
    - (optional)`$ ./manage.py sqlmigrate polls 0001`
        - 不會真的 apply migrations 而是印出對應的 SQL 操作
    - `$ ./manage.py migrate`
        - apply migrations
        - 只會 run 沒有被註解掉的 `INSTALLED_APPS`
- `$ ./manage.py shell`
    - 利用 interactive shell 插入一些測試資料
    ```
    >>> Question.objects.all()
    <QuerySet []>
    >>> from polls.models import Question, Choice
    >>> from django.utils import timezone
    >>> q = Question(question_text = "What's up?", pub_date=timezone.now())
    >>> q.save()
    >>>  Question.objects.all()
    <QuerySet [<Question: Whats up?>]>

    # create choices
    >>> q.choice_set.all()
    <QuerySet []>
    >>> q.choice_set.create(choice_text='Not Much', votes=0)
    <Choice: Not Much>
    >>> q.choice_set.create(choice_text='The sky', votes=0)
    <Choice: The sky>
    >>> q.choice_set.all()
    <QuerySet [<Choice: Not much>, <Choice: The sky>]>
    >>> q.choice_set.count()
    >>> 
    ```

- `$ ./manage.py createsuperuser`
- polls/admin.py
    - register polls under admin
 
---

## part 3
- more views in `polls/views.py`
    - index (already exists)
    - detail
        - 404
    - results
    - vote
    ```=python
    from django.shortcuts import get_object_or_404, render

    # Create your views here.
    from django.http import HttpResponse
    from .models import Question
    
    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        # render() technique in tutorial part 3
        context = {'latest_question_list': latest_question_list}
        return render(request, 'polls/index.html', context)
    
    def detail(request, question_id):
        # get_object_or_404 instead of a try catch in tutorial part 3 
        question = get_object_or_404(Question, pk=question_id)
        return render(request, 'polls/detail.html', {'question': question})
    
    def results(request, question_id):
        response = "You're looking at the results of question %s."
        return HttpResponse(response % question_id)
    
    def vote(request, question_id):
        return HttpResponse("You're voting on question %s." % question_id)
    ```
- template
    > By convention DjangoTemplates looks for a “templates” subdirectory in each of the INSTALLED_APPS.
    - removing hardcoded URLs
    - polls/templates/polls/index.html
    ```=html
    {% if latest_question_list %}
        <ul>
            {% for question in latest_question_list %}
                <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
            {% endfor %}
        </ul>
        {% else %}
        <p>No polls are available.</p>
    {% endif %}

    ```
    
    - polls/templates/polls/detail.html
    
    ```=html
    <h1>{{ question.question_text }}</h1>
    <ul>
        {% for choice in question.choice_set.all %}
            <li>{{ choice.choice_text }}</li>
        {% endfor %}
    </ul>

    ```
    
- more corresponding url rules in `polls/urls.py`
    - Namespacing add `app_name = 'polls'`
- short cuts of writing views
    - render() instead of HttpResponse()
    - get_object_or_404()

---

## part 4
