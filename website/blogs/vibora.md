# Vibora

## Faster Flask?

**Posted by Joe Blankenship on August 07, 2018**

I was working with Flask for my latest project when I became curious as to what other frameworks were out there. Looking at async frameworks, I found a lot of interesting solutions, but was blown away by what I found in Vibora's claims. Despite it being a beta, its speeds and simplicity made it an impressive async alternative to Flask. However, the documentation and framework still have a ways to go.

## The Basics

As demonstrated in their documentation, setting up a basic app is fairly simple and very familiar.

```python
from vibora import Vibora, Response

app = Vibora()


@app.route('/')
async def home():
    return Response(b'Hello world')


if __name__ == '__main__':
    app.run(debug=True)
```

I also found it handy for serving out static files.

```python
from vibora.static import StaticHandler

app = Vibora(
    static=StaticHandler(
        paths=['/data'],
        host='data.thejoeblankenship.com',
        url_prefix='/data',
        max_cache_size=1 * 1024 * 1024
    )
)                       
```

It even comes with some clever router strategies.

```python
from vibora import Vibora
from vibora.router import RouterStrategy

app = Vibora(router_strategy=RouterStrategy.STRICT)
```

## Issues

Data validation and event handling also seem to work well, but documentation and functional examples of templating and testing fall short (there are examples on GitHub, just not fully explained in the documentation). Additionally, there is not a solid way to deploy a Vibora app outside of using docker.

## Conclusion

It's very exciting to see an async framework with such promise, but it has a ways to go before it can gain the popularity of Flask or Django (though slow as they may be). I look forward to coming back and revisting this framework and blog post with updates in the future.
