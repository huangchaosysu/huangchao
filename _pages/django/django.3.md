---
layout: single
permalink: /django/firstapp2/
title: "第一个Django应用-2"
sidebar:
  nav: "django"
toc: true
---

### 配置数据库
<p>编辑<code>mysite/settings.py</code>，这就是一个普通的python文件。里面包含了Django的各种配置项。默认情况下，有关数据库的配置是SQLite，对于数据库新手或者没有安装数据库的人来说，这是一个不错的选择。python自带了SQLite，因此不需要我们自己安装。我们的教程里，会使使用Mysql，因为mysql是现在最流行的开源数据库。有关数据库的配置项都保存在DATABASES这个配置项中。</p>
<pre>
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mysite',
        'USER': 'root',
        'PASSWORD': 'root',
        'HOST': 'localhost',
        'PORT': '3306',
    },
}
</pre>
<p>default是默认的数据库配置名称，ENGINE是数据库的类型，可选的有'django.db.backends.sqlite3', 'django.db.backends.postgresql', 'django.db.backends.mysql', or 'django.db.backends.oracle'，除了这几个还有其他的第三方的可选，我们一般也不会用到，这里就不多说。NAME是要链接的数据库的名字，USER为数据库的用户名，PASSWORD为数据库的密码，HOST为数据库的ip或者域名，PORT是数据库使用的端口。需要注意的是，本示例需要数据库账户有在数据库上运行<code>create database</code>的权限，如果你不是很确定，那么直接使用数据库的root账户吧</p>
### INSTALLED_APPS
<p>留意一下INSTALL_APPS这个配置项，这个列表保存了所有在Django项目中已经激活的app列表。一个app只有被激活以后才能对外提供服务。请确保polls也在这个列表中。这个列表默认会有一些app，这些都是Django自带的一些通用功能的app。在前面的章节提到的<code>python3 manage.py migrate</code>这条命令会搜索INSTALLED_APPS中所有的app，并根据这些app定义的model创建数据库表</p>
<pre>
INSTALLED_APPS = [
    "polls.apps.PollsConfig",  # "这一行只填polls也是可以的"
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
</pre>
### 定义Model
<p>接下来要讲解的是如何定义Model，也就是如何定义数据库的表结构</p>
<p>在示例的polls这个app中，我们将会定义2个Model，分别是<code>Question</code>和<code>Choice</code>。一个Question对象包含一个问题属性和一个发布日期属性。Choice对象包含一个文本属性和一个投票计数属性。每个Choice都会关联到一个Question。编辑<code>polls/models.py:</code></p>
<em>polls/models.py</em>
<pre>
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
</pre>

<p>这段代码很容易理解，我们要定义的2个Model都继承<code>models.Model</code>，每个Model都有若干类属性，每个属性代表了一个数据库字段。每个属性都是一个Field类型的实例对象，Django提供了多种Field类型来对应不同的数据类型。例如：CharField代表字符串类型的数据，DateTimeField对应的是时间类型的数据。</p>
<p>每一个Field实例的名字(question_next，pub_date)就是在python代码中访问该属性的名字，也是数据库的列名。可以通过Field字段的第一个位置参数来自定义一个对人友好的名字,这个名字在后面讲解admin的时候会用到。另外，某些Field类型会有必填参数，比如<code>CharField</code>的<code>max_length</code>这个属性就是必填的。</p>
<pre>在上面的例子中，我们定义了一个外键，这是通过ForeighKey这个Field来实现的。</pre>

### 激活Model
<p>上面我们有一个Model的示例，Django会根据Model定义做2件事情:</p>
<ul>
    <li>根据当前app定义的Model来创建数据库表</li>
    <li>创建访问Choice和question这两个数据库表带API</li>
</ul>
<p>再次确保把polls添加到INSTALLED_APP中，运行下一条命令，<code>$ python3 manage.py makemigrations polls
</code>，应该看到类似的输出:</p>
<pre>
Migrations for 'polls':
  polls/migrations/0001_initial.py:
    - Create model Choice
    - Create model Question
    - Add field question to choice
</pre>
<p>makemigrations这个命令的作用是告诉Django我们定义的Model已经修改过了，需要把修改内容保存为一个新的migration</p>
<p>migration: 在Django中，migration的概念是用来管理Model的各个版本的，有点类似于版本管理器的作用。在polls/migrations目录里面存放了所有版本的migration，有兴趣的童鞋可以打开文件看一下里面的内容, 编号就是文件名前面的数字。</p>
<p>使用<code>sqlmigrate</code>这个命令可以看到Django是如何创建或者更新数据库表的(本质上是通过sql语句)。</p>
<p><code>$ python3 manage.py sqlmigrate polls 0001</code>，0001为migration的编号，运行这条命令可以看到下面类似的输出:</p>
<pre>
BEGIN;
--
-- Create model Choice
--
CREATE TABLE `polls_choice` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `choice_text` varchar(200) NOT NULL, `votes` integer NOT NULL);
--
-- Create model Question
--
CREATE TABLE `polls_question` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `question_text` varchar(200) NOT NULL, `pub_date` datetime(6) NOT NULL);
--
-- Add field question to choice
--
ALTER TABLE `polls_choice` ADD COLUMN `question_id` integer NOT NULL;
ALTER TABLE `polls_choice` ALTER COLUMN `question_id` DROP DEFAULT;
CREATE INDEX `polls_choice_7aa0f6ee` ON `polls_choice` (`question_id`);
ALTER TABLE `polls_choice` ADD CONSTRAINT `polls_choice_question_id_c5b4b260_fk_polls_question_id` FOREIGN KEY (`question_id`) REFERENCES `polls_question` (`id`);

COMMIT;
</pre>
<p>有几点事情有必要拿出来说一下</p>
<ul>
    <li>具体的输出根据使用的数据库会有所不同，上例的输出是数据库选用mysql时的输出</li>
    <li>数据库的表名默认是由app的名字(polls)与Model(choice, question)的小写名字合并而来</li>
    <li>Django会自动添加一个primary key的字段，id，该字段是int类型并且是自增的</li>
    <li>默认情况下Django会把'_id'添加到外键名称的末尾来作为外键的列名，本例为'question_id'</li>
    <li>Django会根据选用的数据库类型自动的把Model里面的Field映射到该数据库与之匹配的类型，并生成对应的sql语句，完全不用人工干预</li>
    <li>sqlmigrate这个命令并不会真正的创建对应的表，它只是把Django在生成数据库表时要执行的sql语句输出到屏幕上，供我们检查或者手动运行</li>
</ul>
<p>现在，运行migrate这个命令来真正的创建数据库表</p>
<pre>
$ python3 manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
</pre>
<p>migrate这个命令会把所有未执行过的migration同步到数据库。后续随着开发的推进，我们还可以通过migration来自动的修改数据库。要完成这个任务只需三步
</p>
<ul>
    <li>修改Model</li>
    <li>运行<code>python3 manage.py makemigrations</code>来生成一个新的migration</li>
    <li>运行<code>python3 manage.py migrate</code>将修改同步到数据库</li>
</ul>
### 使用API
<p>运行<code>$ python3 manage.py shell</code>进入python的交互模式，在交互模式下我们可以试验一下Django提供的API</p>
<p>注意，这里我们是通过manage.py来打开python的交互模式, 有没有发现什么不一样的地方？是的。平时我们都是直接使用<code>python3</code>这个命令来进入python的交互模式，那这里为什么不这样做呢？原因在于，要想使用Django提供的一些特性，需要设置一些环境变量。使用<code>python3 manage.py shell</code>可以让Django自动帮我们完成对环境变量的设置工作。如果你就是不想Django自动来完成的话，那也没问题，只需把DJANGO_SETTINGS_MODULE环境变量的值指向mysite.settings，并且在python交互模式下运行下面的命令就可以正常使用Django了</p>
<pre>
>>> import django
>>> django.setup()
</pre>
<p>设置好了运行环境，就来体验一下Django提供的api吧。注意观察一下数据库内容的变化哦</p>

```
>>> from polls.models import Question, Choice   # Import the model classes we just wrote.

# polls_question表还没有数据
>>> Question.objects.all()
<QuerySet []>

# 创建一个Question对象，并存入polls_question表
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# 必须显示调用save才能触发数据库的保存动作
>>> q.save()

＃ 自动生成的id
>>> q.id
1

# 通过python的属性来访问Model的Field
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# 修改属性的值
>>> q.question_text = "What's up?"
>>> q.save()

# 选择数据库中的全部数据
>>> Question.objects.all()
<QuerySet [<Question: Question object>]>
```
<p>上面最后一个api调用的结果为<code><QuerySet [<Question: Question object>]></code>， 这种输出用户很难看懂，我们可以给对应的Model定义__str__()方法来解决这个问题</p>
<em>polls/models.py</em>
<pre>
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
</pre>
<p>我们可以像给普通的python类定义方法一样给Model类定义方法。接下来，我们给Question定义一个方法</p>
<em>polls/models.py</em>
<pre>
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
</pre>
<p>保存修改并重新运行<code>python3 manage.py shell</code>, 尝试下面的api调用</p>

<pre>
>>> from polls.models import Question, Choice

# __str__()起作用了
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django提供了丰富的数据库查询接口
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# 查询今年发布的问题
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# 尝试查找不存在的id
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Django中，根据主键查询既可以使用id＝1，也可以使用pk＝1， pk就代表主键
>>> Question.objects.get(id=1)
<Question: What's up?>

>>> Question.objects.get(pk=1)
<Question: What's up?>

# 尝试调用自定义方法
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# 下面的代码演示了Django中的外键查询
>>> q = Question.objects.get(pk=1)

# 查询与question关联的所有choice，当前还没有，因此为空
>>> q.choice_set.all()
<QuerySet []>

# 调用create创建3个choice，create的作用为创建choice并保存到数据库。
# 这里注意到我们在从question访问由外键关系所关联的choice对象时使用了choice_set这个属性
# 在Django中，反向访问外键关联默认的属性名字为Model名(这里是choice)后面加'_set',
# 本例中，外键关系是由Choice指向Question的，因此反向访问(question->choice)就是choice_set
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# 正向访问外键关联的对象，这里是从choice访问其关联的question
>>> c.question
<Question: What's up?>

# 从question访问choice
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# Django提供的查询api，其查询条件可以多级嵌套，下面这个例子是选取关联的question的发布日期的年份为今年的choice
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# 删除
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
</pre>