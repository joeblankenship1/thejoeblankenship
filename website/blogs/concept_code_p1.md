# Conceptualizing Code - Part 1

## Python, PDFs, and MongoDB

**Posted by Joe Blankenship on September 28, 2017**

In this blog series, I want to go through a process of thinking about “how I think about” code and coding. The impetus for this series primarily comes from the nature of how I learned to code outside of a structured, collaborative classroom setting. Despite my years of work in developing software and using that software to solve real-world issues, I often have trouble talking to other developers and engineers about how I conceptualize code.

I started coding in GW-BASIC when I was very young, producing a couple playable games on my Dad’s computer. I then worked with C++ and a number of assembly languages while going to college for engineering, but after graduation my focus intensified on hardware and electronics (design and manufacturing) which took me away from developing software. While using and maintaining my engineering skills (both professionally and as a hobby), I’ve always wanted to properly relearn coding; not just as a necessity for projects, but as something I truly enjoy doing.

Over the last couple years, I’ve made a concerted effort to improve my skills through books, podcasts, online courses, developer community meetings: anything to keep improving my homegrown affinity for coding. With the recent “data science” boom, I found an incredibly rewarding way to combine my years of analytical, engineering, and coding skills through experimenting with and hacking on data and systems with Python.

For the first part in this series, we are going to look at ***how*** I look at a problem, formulate a solution, and produce the code I need to accomplish my list of requirements. The goal is two-fold. First, this will help me address problems in how I go about planning and executing a software solution that I then have to communicate to others. Secondly, this will hopefully help other people who are just starting out with coding say, “if this guy can do it, surely I can.”

## Getting Started

I have an application I want to build which will allow social scientists to perform qualitative analysis. One of the core requirements for the application is to import a number of data formats, one at a time or in bulk, as part of an ETL (extract transform load) pipeline. Ultimately, this functionality will be part of a Flask application that the scientists can use through their web browser.

So two initial questions: what are the data types I need to import and where am I going to put the data. For this series, I will store the data in MongoDB (PostgreSQL was a very close second; this was based on other requirements needed for the overall application) and I will initially focus on importing PDF files since they often tend to be challenging.

If you do not have MongoDB installed, you will need to do that now ([click here for tutorial](https://docs.mongodb.com/manual/installation/ "https://docs.mongodb.com/manual/installation/")). Then, you will need to generate a Python virtual environment for the libraries specific to this blog series.

> Note: I am using Ubuntu 16.04 and Python3. If you are running Windows or MacOS, you will have to find the equivalent commands.

```bash

# navigate to the directory into which you wish to build the virtual environment, then...
python3 -m venv name_of_environment

# activate the virtual environment
cd name_of_environment
source bin/activate

# then install some libraries
pip3 install pdfx pymongo jupyter

```

Notice we also installed Jupyter Notebooks. At this stage in the project, I generally like to experiment with the libraries, read the documentation, and structure my pseudocode in an easy-to-use environment that will also allow me to export to a ***.py*** file. Keep in mind, the tools we’re building now are going to have to integrate into the chosen UX/UI framework that the scientists will be using, so reading up on Flask functionality at this point is also very important.

We are now mostly ready to conceptualize how to extract data from PDFs.

## PDFs and PDFx

[PDFx](https://github.com/metachris/pdfx "https://github.com/metachris/pdfx") is a project by Chris Hager for the extraction of text, references, and metadata from PDFs. I chose to use this as it simplifies much of the PDFMiner libraries quirks and give us a very structured, pretty output for our MongoDB records.

What we need to do initially is to create a PDF object using this library in order to then extract the metadata, references, and text from our PDF(s).

```python

# import library
import pdfx

# create a pdf object for a single file
pdf = pdfx.PDFx("filename.pdf")

# extract the metadata from the pdf object
metadata = pdf.get_metadata()

# extract the references from the pdf
# create a python dictionary of the references
reference_dict = pdf.get_references_as_dict()

# later if you wanted to crawl from this PDF via its references
# it can download PDFs from reference hyperlinks
# pdf.download_pdfs("temp_down_folder/")

```

Now at this point, we've done what we've set out to do, but the text looks like a mess when we attempt to extract it. We need to better structure our output before we put it into MongoDB. At a minimum, we need to group the text by heading and paragraph as to make qualitative coding, searching, and analysis much easier.

```python

# extract the text from the PDF
# replace the return characters with nothing creating one long string
# split the string at the form feed characters
text = pdf.get_text()
text_to_mongo = text.replace('\n', '').split("\x0c")

```

This should result in a much improved document structure that can then be processed using natural language processing/machine learning (which is a requirement we'll build towards later, but plan for now). The above code will be further abstracted for use in functions and classes within our Flask application, so it is important that we think about how this single file subroutine may have to be manipulated for bulk uploads (including async and threading considerations for large PDFs with images and tables).

Now, we are ready to conceptualize how we are going to push this into MongoDB.

## MongoDB and PyMongo

[MongoDB](https://www.mongodb.com/ "https://www.mongodb.com/") is a document-oriented database classified as NoSQL. It stores records in its "schema-less" JSON (called BSON) format. Since we're using Flask, we'll eventually interact with MongoDB through a RESTful API using [Eve](http://python-eve.org/foreword.html "http://python-eve.org/foreword.html"), but for now we'll be using [PyMongo](https://api.mongodb.com/python/current/ "https://api.mongodb.com/python/current/") directly to understand how to interface between our extracted data and MongoDB.

What we need to do initially is create a MongoDB object through which we can push our PDF metadata, references, and text as a single document into the database.

```python

# import MongoClient from the PyMongo library, datetime, and pprint
from pymongo import MongoClient
import datetime
import pprint

# insert documents (stored as dictionaries in BSON (JSON) format (UTF-8))
# notice we can create the schema we want ad-hoc for our document
post = {
    "metadata": metadata,
    "text": text_to_mongo,
    "references": reference_dict,
    "import_date": datetime.datetime.utcnow(),
    "tags": ["pdf"]
}

# if you have already created a database or need to create a new database
db = client.name_of_database

# create an object for your PDF data, then insert it into the database
posts = db.posts
post_id = posts.insert_one(post).inserted_id

# now check the output for your new MongoDB document
pprint.pprint(posts.find_one())  # search with dict keys or "_id" for Mongo UID

```

At this point, we can now see that our PDF output from before can be queried from our database. MongoDB allows us a lot of flexibility, but this flexibility needs to be closely regulated through our API once we get ready to move our conceptual ETL pipeline into our Flask application.

## Final Thoughts

A lot of the times, the hardest part of this process is going from the "that would be a great idea" stage to the pseudocode stage of planning your application's sometimes numerous functionalities and dependencies. And it’s really about having the confidence to mess up, recover from those mistakes, and if need be reaching out to more experienced developers for help: everyone conceptualizes their projects differently. One of the reasons I love Python is that every developer I’ve met has been more than open to pointing me in the right direction. It’s okay to ask naive questions on Stack Overflow; it’s how you learn.

Small things like this also help you overcome “impostor syndrome” when talking/working with other developers. Showing that you’ve done some work before approaching others with questions always goes over better. Moreover, if you have code, they’ll identify you as a coder and that goes along way in building great working relationships and eventually your reputation as a developer/data scientist.

In the following sections of this series, we’ll further abstract what we've done here into Pythonic functions and classes. We will then begin placing them into our Flask application as part of a functionality that will allow us to submit, query, and recall documents.
