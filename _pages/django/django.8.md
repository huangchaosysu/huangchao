---
layout: single
permalink: /django/firstapp7/
title: "第一个Django应用-7"
sidebar:
  nav: "django"
toc: true
---

<p>本章节，我们将介绍如何把polls变成一个standalone的Python包，以便在我们的其他项目中重用其代码或者分享给其他开发者</p>
<p>Python这门语言能够发展到今天这个程度，很大程度上是因为代码复用。<a class="reference external" href="https://pypi.python.org/pypi">The Python Package Index (PyPI)</a>上面有大量的可以直接拿过来应用到我们自己代码里的python包。<a class="reference external" href="https://www.djangopackages.com">Django Packages</a>上面提供了许多可以复用的Django app。就连Django自身也只不过是一个可复用的Python包。因此，代码复用对于Python来说是相当重要的。代码重用可以减少我们的编码量，在写程序时我们只需要编写那些针对特定业务的代码逻辑，而其他的通用逻辑可以使用已有的代码来完成。注意，并不是所有的功能都有现成的代码，这里只是说在有现成代码可用的情况下可以这么做</p>
<p>接下来给大家演示如何把app变成可复用的。完成前面的教程后，我们的代码目录应该类似于下面这个结构</p>

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
    polls/
        __init__.py
        admin.py
        migrations/
            __init__.py
            0001_initial.py
        models.py
        static/
            polls/
                images/
                    background.gif
                style.css
        templates/
            polls/
                detail.html
                index.html
                results.html
        tests.py
        urls.py
        views.py
    templates/
        admin/
            base_site.html
```
<p>在现在这个代码结构下，如果是我们自己想要复用polls这个app，那么可以把这个目录直接copy到其他项目中就可以用了。但是这还不能算真正的复用。我们的终极目标是做到可以像安装其他包一样能够使用<code>pip3 install xxx</code>来安装我们的app</p>

### 安装依赖

<p>后面的过程需要依赖一些工具，比如pip3(python2为pip)、setuptools, 关于如何安装pip3和setuptools，请参考<a href="https://packaging.python.org/installing/#install-pip-setuptools-and-wheel">install-pip-setuptools-and-wheel</a></p>

### 打包APP
<p>所谓的打包app其实是把我们的app代码按照python packaging的要求来保存。python packaging是指按照一定的格式来把代码打包，这样就能够通过pip的方式来快速方便的安装了，下面是打包app的步骤</p>

* 在Django项目外面为polls这个app创建一个父目录，例如django-polls(在起名时最好查一下pypi以确定是否有同名的包，以防后面出现名字冲突)
* 把polls这个目录移到django-polls目录下
* 新建文件django-polls/README.rst并加入以下内容

<pre>
=====
Polls
=====

Polls is a simple Django app to conduct Web-based polls. For each
question, visitors can choose between a fixed number of answers.
Detailed documentation is in the "docs" directory.

Quick start
1. Add "polls" to your INSTALLED_APPS setting like this::

    INSTALLED_APPS = [
        'polls',
    ]

2. Include the polls URLconf in your project urls.py like this::

    url(r'^polls/', include('polls.urls')),

3. Run `python manage.py migrate` to create the polls models.

4. Start the development server and visit http://127.0.0.1:8000/admin/
   to create a poll (you'll need the Admin app enabled).

5. Visit http://127.0.0.1:8000/polls/ to participate in the poll.
</pre>

<p>从以上的内容可以看出该文件的内容为使用说明</p>


* 新建许可文件jango-polls/LICENSE，在该文件内加入许可内容。有关license的选择读者可以自行google或百度，常用的许可有apache、bsd许可等
* 新建setup.py。该文件提供了pip安装某个package需要用到的详细信息

<pre>
import os
from setuptools import find_packages, setup

with open(os.path.join(os.path.dirname(__file__), 'README.rst')) as readme:
    README = readme.read()

\# allow setup.py to be run from any path
os.chdir(os.path.normpath(os.path.join(os.path.abspath(__file__), os.pardir)))

setup(
    name='django-polls',
    version='0.1',
    packages=find_packages(),
    include_package_data=True,
    license='BSD License',  # example license
    description='A simple Django app to conduct Web-based polls.',
    long_description=README,
    url='https://www.example.com/',
    author='Your Name',
    author_email='yourname@example.com',
    classifiers=[
        'Environment :: Web Environment',
        'Framework :: Django',
        'Framework :: Django :: X.Y',  # replace "X.Y" as appropriate
        'Intended Audience :: Developers',
        'License :: OSI Approved :: BSD License',  # example license
        'Operating System :: OS Independent',
        'Programming Language :: Python',
        # Replace these appropriately if you are stuck on Python 2.
        'Programming Language :: Python :: 3',
        'Programming Language :: Python :: 3.4',
        'Programming Language :: Python :: 3.5',
        'Topic :: Internet :: WWW/HTTP',
        'Topic :: Internet :: WWW/HTTP :: Dynamic Content',
    ],
)
</pre>
<p>关于setup.py这个文件的详细内容读者可参考<a href="https://setuptools.readthedocs.io/en/latest/">setuptools-doc</a></p>


* 创建MANIFEST.in。默认情况下只有python文件会被打包，如果需要包含其他非python文件，那么需要添加一个名为MANIFEST.in的文件并把要包含的内容写进去

<pre>
include LICENSE
include README.rst
recursive-include polls/static *
recursive-include polls/templates *
</pre>
* 最后一步是打包。在django-polls目录下运行命令<code>python3 setup.py sdist</code>，这条命令会创建一个dist目录和一个压缩文件django-polls-0.1.tar.gz

<p>更加详细的有关如何打包和发布python项目请参考<a class="reference external" href="https://packaging.python.org/en/latest/distributing.html">Tutorial on Packaging and
Distributing Projects</a></p>

### 发布app
<p>完成前面的步骤，我们的app应该已经打包好了，剩下的就是发布了。有以下几种渠道</p>
<ul>
    <li>通过邮件发送给朋友</li>
    <li>发布到个人网站</li>
    <li>发布到PyPi(the Python Package Index)<a href="https://packaging.python.org/distributing/#uploading-your-project-to-pypi"></a></li>
</ul>