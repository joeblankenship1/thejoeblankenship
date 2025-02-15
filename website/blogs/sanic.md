# Sanic

## Going faster with Async Web Development

**Posted by Joe Blankenship on January 01, 2019**

## Async and Beyond

My most recent re-introduction to [Sanic](https://sanicframework.org/ "https://sanicframework.org/") was very pleasant. Since its initial release, it has become much easier to use and at this time is the most consistently supported async microframework for Python. Despite my still nascent experience with async, Sanic's demos and documentation made adapting my knowledge of Flask very easy. Sanic is surely going to be my go-to framework for fast data APIs moving forward.

## A Simple Data API

Building a simple data API can be accomplished in a few lines of code.

```python

from sanic import Sanic

# create an app object
app = Sanic(__name__)
# serve your static data file at /
# you can stipulate virtual host with host='url'
# if your data is large, use stream_large_files=True
app.static('/', 'census_2010_ky.json', name='census')
# run the app
app.run(host="0.0.0.0", port=8080)

```

I can now serve out processed data files for use in a front-end application (especially spatial data files which can grow to very large sizes). If you want to build a service for your database, [Gino](https://github.com/fantix/gino "https://github.com/fantix/gino") is an option. In fact, Sanic has several [extensions](https://sanic.readthedocs.io/en/latest/sanic/extensions.html "https://sanic.readthedocs.io/en/latest/sanic/extensions.html") noted in their official documentation for security, testing, and more.

## A Simple Website

Building a simple website is also a breeze with Sanic.

```python

from sanic import Sanic
from sanic.response import html
from jinja2 import Environment, PackageLoader, select_autoescape

# define the environment for the Jinja2 templates
env = Environment(
    loader=PackageLoader('main', 'templates'),
    autoescape=select_autoescape(['html', 'xml', 'tpl'])
)


# a function for loading an HTML template from the Jinja environment
def template(tpl, **kwargs):
    template = env.get_template(tpl)
    return html(template.render(kwargs))


# create the Sanic app and serve it statically
app = Sanic(__name__)
# if you have static files in /static directory, use the below statement
# app.static('/static', './static')


# define our function for our homepage
@app.route('/')
async def home(request):
    greeting = 'Hello, Sanic!'
    link = 'https://sanic.readthedocs.io/en/latest/'
    return template(
        'index.html',
        title='Sanic Website - Demo',
        greeting=greeting,
        button=link
    )


# define our function for our homepage
@app.route('/about')
async def about(request):
    greeting = 'About'
    link = 'https://github.com/huge-success/sanic'
    return template(
        'about.html',
        title='Sanic Website - Demo',
        greeting=greeting,
        link=link
    )


# run the main.py on http://localhost:8000
if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8000)

```

This is obivously very similar to Flask in both its structure and simplicity. I had a bit of issue with jinja2-sanic, but was able to get Jinja2 working with my templates extremely well.

Overall, for the purposes of building fast data APIs and responsive web applications, Sanic is becoming my default microframework. You can find my source code [here](https://github.com/joeblankenship1/homepage_sanic "https://github.com/joeblankenship1/homepage_sanic").

## References

* [Sanic Homepage](https://sanicframework.org/ "https://sanicframework.org/")  
* [Sanic Documentation](https://sanic.readthedocs.io/en/latest/ "https://sanic.readthedocs.io/en/latest/")  
* [Sanic on GitHub](https://github.com/huge-success/sanic "https://github.com/huge-success/sanic")  
