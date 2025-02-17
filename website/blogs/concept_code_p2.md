# Conceptualizing Code - Part 2

## Modules and Eve

**Posted by Joe Blankenship on November 16, 2017**

First, if you haven't read part 1, please proceed [here](https://thejoeblankenship.com/blogs/concept_code_p1.html "https://thejoeblankenship.com/blogs/concept_code_p1.html") first and then come back.

In the first part, we built very basic scripts for processing PDFs and pushing the output into MongoDB. However, that is not very efficient if we have more than one PDF or we're importing, processing, and manipulating the data through a dashboard, API, or webform; we need to put our basic scripts into a set of callable methods. I've opted to create two classes: one for extracting the PDF information and one for loading that data into the MongoDB collection for PDFs.

The goal of this is to make our code as reusable as possible since this will later be used in our application.

## From Snippets to Modules

> Note: I am using Ubuntu 16.04 and Python3. If you are running Windows or MacOS, you will have to find the equivalent commands.

After activating your virtual environment, open up Jupyter Notebook and begin to examine the sections related to PDF import and extraction. We have to import the PDFx library, create a pdf object, and then extract the metadata, references, and main text. However, we need to be able to send any PDF through this process so that we can populate our database.

```python

# be inside of your virtual environment's directory (e.g., name_of_environment)
import pdfx

class PDFExtract:
    '''A class that uses PDFx to
    extract metadata, references, and text'''
    def import_metadata(self):
        pdf = pdfx.PDFx(self)
        metadata = pdf.get_metadata()
        return metadata

    def import_references(self):
        pdf = pdfx.PDFx(self)
        reference_dict = pdf.get_references_as_dict()
        return reference_dict

    def import_text(self):
        pdf = pdfx.PDFx(self)
        text = pdf.get_text().replace('\n', '').split("\x0c")
        return text

```

I have taken my snippets from part 1 and turned them into methods inside of a larger class dedicated to extracting the data we need from PDFs.

Now we can call our class methods for extracting the different sections of the PDF that we need.

```python

# return the metadata for a single file
pdf_file = "pdf_file.pdf"
PDFExtract.metadata(pdf_file)

```

We now have to push the PDF data into MongoDB.

```python

from pymongo import MongoClient
import datetime

class PDFLoad:
    '''A class used to load PDFs into MongoDB'''
    def mongo_load_pdf(self):
        client = MongoClient()
        db = client.mongo_pdf
        posts = db.posts
        post = {
            "metadata": PDFExtract.import_metadata(self),
            "text": PDFExtract.import_text(self),
            "references": PDFExtract.import_references(self),
            "import_date": datetime.datetime.utcnow(),
            "tags": ["pdf"]
        }
        posts.insert_one(post)

```

Once again, our snippets now constitute a method inside of a class dedicated to loading our extracted PDF data into MongoDB.

Now we can simply pass a single command to load our PDF file into Mongo DB.

```python

# Push a single PDF into MondoDB 'mongo_pdf' collection
PDFLoad.mongo_load_pdf("pdf_file.pdf")

```

Now, using the **%timeit** magic in Jupyter, executing this operation takes way to long! Before we put this into our Flask app, we have to analyze our code to determine the bottlenecks for our current design.

In examination of the above code, each method in my PDFExtract class has to load a new PDF object each time they run. The PDFLoad class then has to call to PDFExtract for each of its methods. This is far too much back and forth for a single file input and will be too long for multiple imports. So lets combine our two classes into a single class.

```python

#!/usr/bin/env python3
"""
PDFExLoad
ExLoader
"""

import pdfx
from pymongo import MongoClient
import datetime

class PDFExLoad:
    '''A class that uses PDFx to
    extract metadata, references, and text
    we then load the output into MongoDB'''
    def ExLoader(self):
        pdf = pdfx.PDFx(self)
        client = MongoClient()
        db = client.mongo_pdf
        '''Extract the metadata, text, and references'''
        metadata = pdf.get_metadata()
        text = pdf.get_text().replace('\n', '').split("\x0c")
        reference_dict = pdf.get_references_as_dict()
        '''Take the extracted data and push it to MongoDB'''
        posts = db.posts
        post = {
            "metadata": metadata,
            "text": text,
            "references": reference_dict,
            "import_date": datetime.datetime.utcnow(),
            "tags": ["pdf"]
        }
        posts.insert_one(post)

if __name__ == "__main__":
    import sys
    PDFExLoad.ExLoader(str(sys.argv[1]))

```

I have combined the extractor and loader classes together into a single class that has a streamlined method for single PDF extraction and loading. I've also added proper styling to my module in case I want to reuse my code later as a stand-alone program.

This results in a 300% increase in speed! With some async magic in Flask, I can now call to MongoDB for each element extracted from the PDF.

I still feel this is bit slow, so I'll revisit my implementation of the PDFx library later; possibly opting to user the PDFMiner library directly to alleviate the remaining speed optimization issues. You can save the above file as **pfdexload.py** if you wish to use it as a stand-alone program.

## Flask API with Eve

[Flask](http://flask.pocoo.org/ "http://flask.pocoo.org/") "is a microframework for Python based on Werkzeug and Jinja 2." Thankfully for us, it makes building APIs, microservices, and web apps a breeze.

Inside of your activated virtual environment, install Flask and the dependencies you will need to build the API.

```python

# Install Flask, Eve, PyMongo
# be inside of your virtual environment's directory (e.g., name_of_environment)
pip3 install eve Flask flask-pymongo

```

In addition to Flask and PyMongo, we installed [Eve](http://python-eve.org/ "http://python-eve.org/") which is an open source Python REST API framework that will work securely and efficiently with MongoDB. We can now proceed in creating our initial Flask framework for the Eve API.

```python

# for Mac and Windows users, you will have to find the equivalent commands for
# the terminal/command prompt respectively.
mkdir app
mkdir app/api
touch app/api/eve_server.py
touch app/api/settings.py

```

Now that we have a file folder structure and the API files, we need to code and configure the files with our API instructions.

```python

# eve_server.py

from eve import Eve  # import the Eve library from your virtual environment

app = Eve()  # create an Eve object named app

# this statement allows this file to run as an independent Python program
if __name__ == '__main__':
    app.run(debug=True)

```

```python

# settings.py

# Mongo Settings commented out as we will tell Eve where to look for Mongo below
# This will be part of our Flask app configuration later
# MONGO_HOST = 'localhost'
# MONGO_PORT = '27017'
# MONGO_USERNAME = ''
# MONGO_PASSWORD = ''

URL_PREFIX = 'api'  # this will proceed 'localhost:5000'
MONGO_DBNAME = 'mongo_pdf'  # the database in MongoDB we've been using
# here we will identify the 'posts' collection and the schema we want to output
# from our Eve API
DOMAIN = {
    'posts': {
        'schema': {
            'metadata': {'type': 'string'},
            'text': {'type': 'string'},
            'references': {'type': 'string'},
            'import_date': {'type': 'integer'},
            'tags': {'type': 'string'}
        }
    }
}
X_DOMAINS = 'http://localhost:8080'  # this URL is allowed to access our API
HATEOAS = False  # allows/denies dynamic navigation of API
PAGINATION = False  # set to integer to limit the number of output items/page

```

For more details on Eve API setup, refer to the [Eve documentation](http://python-eve.org/ "http://python-eve.org/").

Now we can go to our Terminal and run our server.

```bash

# Run our Eve API server
python eve_server.py

```

This should give us a prompt that our API service is running on localhost port 5000. We can now use **curl** to test our API output. If you don't have curl or cannot use it, then you can go to [http://localhost:5000/api/posts](http://localhost:5000/api/posts "http://localhost:5000/api/posts") to view the API output.  

> Note: I wouldn't do this if your database is large; it could lock up your browser. You can adjust your pagination setting to partially resolve this issue, but you should use curl if possible.

```bash

curl -g http://localhost:5000/api/posts\?where\=\{\"tags\":\"pdf\"\}

```

This should return every document in your MongoDB collection posts with the tag **pdf**. Curl can run some fairly complex queries against our API, so feel free to experiment.

## Final Thoughts

In this section, we've taken our very simple code snippets in Jupyter Notebooks and created an ETL pipeline tool for our PDF data. We then took that data and rendered it through a secure RESTful API. All of this was done with relatively litte code.

Hopefully, this makes you a bit more confident in your ability to take a little bit of knowledge in Python and turn it into action you can take at work, school, or on the weekend when you're geeking out over a rich, new data set. I still have a ways to go in order to make the code above even better by using threading, more efficient class objects/methods, and design patterns based on planned implementation of my pipeline for the service we will be building in part 3 of this blog series.

So if you take anything away from this, just keep in mind that developers and data scientists are always learning and as long as you stay motivated, we'll be learning together.

I also want to credit Kyran Dale as I borrowed from [his book](http://shop.oreilly.com/product/0636920037057.do "http://shop.oreilly.com/product/0636920037057.do") for the Eve API section (I highly recommend it).
