# Teaching Python

## Basics to Data Science

**Posted by Joe Blankenship on November 13, 2018**

## Introduction

> “There is no way I can teach this to someone else; I don’t even know if I know this good enough myself!”

This is what goes through my head every time I talk to anyone who wants to know what I know. Whether this is related to academic work, open-source projects, or occupational tasks, there is always the fear of being called an impostor. Even though I know I can accomplish the things they ask about, for some reason it’s hard for me to translate my scattered mental constructs into an easy-to-understand set of instructions for others. However, over the years I’ve found a few surefire ways of overcoming this self-intimidation among my peers and students: training to teaching.

Training to teach a subject is the best way that I’ve found to learn a subject. For me, teaching what I know is grounded in my desire to learn something and to retain that knowledge. So that is the approach I tend to take when creating a course or preparing to learn a new skill: I learn as if I’m going to become the “expert” in that area of knowledge.

I’ve always loved using the Python programming language. I started using it as part of my work with GIS and spatial analysis in GUI-based tools like ArcGIS and later QGIS. However, it wasn’t until I started doing batch data operations and more complex spatial analysis that I started to really learn the utility of the Python libraries that these GUIs used in the background. Over the years, I became very good as knowing how I use Python to get things done, but not so great as telling others how I did those things.

Fast forward a little over a decade, I’m on my second round of teaching a [Python for Data Science course](https://github.com/bluegrass-devs/python_immersion "https://github.com/bluegrass-devs/python_immersion") to my local Meetup groups and my academic colleagues. Through the iterative processes of going through materials and then meeting with co-learners to go over the materials, I’ve not only found a way to retain my knowledge, but several ways to help others understand those same concepts. What follows are my biggest takeaways from this process.

## The Basics

The biggest thing to remember as someone with experience: the basics for you are not the basics for a newcomer. You as the teacher and co-learner have to understand that in order to help a newcomer build their foundation, you have to learn how you built yours. No two developers or data analysts think the same nor will they work in the same ways. So how do I effectively help someone learn things that have become intuitive to me as a researcher and data geek?

First and foremost: slow down. One thing I’ve learned in teaching is to take time to watch people working through the foundational concepts in Python. This way you ensure they fully understand not only what they are seeing, but that they can apply those concepts to their Python paradigm (e.g., the reasons and use-cases they started learning Python for in the first place). Ask questions and make sure exercises and demos are placed in the right spots as to not overload their ability to learn Python as a cumulative process (e.g., they are learning Python in a manner that connects basic elements into more complex concepts). You’re their guide through this process, so it’s important not to lose them in the maze of your own thoughts.

Also keep in mind that you may not know everything about Python and that is fine so long as you know what you know correctly. If you approach teaching as a process of co-learning, you and the student get to explore the subject together. This does a couple of really interesting things. First, it lets them know that even experienced people are constantly learning. Secondly, you both get to have a shared learning experience that helps them learn problem-solving techniques and social practices in a working environment. These wouldn’t seem to be key skills to learn in a coding class, but they often prove critical in interviews and workplaces where working as part of a team requires a self-reliant person with effective social decorum to ensure tasks are understood fully and accomplished on time.

Keeping the above points in mind, it is important to start small and build in a way that best suits the students (and in some cases yourself). It is okay to build a very ambitious curriculum for a class, but if you’re losing students you have to take a step back and ask why. Flexibility and reflection are critical in teaching as it can help you and the students to meet common goals and more importantly, mutually-shared small victories. Building a simple hello world script may not seem very huge to a teacher, but that may be just what a student needs to say, “I can do this, what’s next!” This means having a course modular enough to adjust direction with scenarios that help them understand how small pieces build towards a bigger picture.

## Tailor the Material

As mentioned above, a well conceived course is only as good as it’s ability to be used by a student. This is why auditing your students (if possible) should be one of the first things done in your class. A well-design survey can help preface your teaching expectations before the students show up for the first day of class. Next, spending the first part of the course getting to know what they expect to get out of the lessons as well as what they may perceive as a challenge will help you better adjust course materials. Often a class is not about teaching a subject to students: it’s about helping them to ask better questions for themselves using the tools, techniques, and methods you offer.

This is where building the course to your strengths meets with the strengths you want to build for them and for yourself. If you understand how the class needs and desires to use what you offer, it is the first step in building a course that you and they will enjoy. This means that you as a teacher will have to initially prepare for rapid revisions in the material and learning goals. There is a saying: “Proper planning and preparation prevents piss-poor performance.” This is especially true of teaching a technology that is constantly evolving as the demands surrounding the technology evolve and directly affect it.

## Feedback

The above teaching tasks depend heavily on your ability to obtain and use feedback from your students and to apply it quickly to ensure they are getting the most from the class. This refers back to two concepts mentioned above: flexibility and modularization.

Flexibility in the face of feedback from the class depends heavily on your depth of knowledge and your ability to hack that knowledge for your students’ benefit. If they are having issues with data structures, make sure you have separate demos and exercise that focus on those challenges outside of the primary scenario for the class, but designed with the intent to bring those learned skills back into the class overall.

This is a part of keeping course materials modular. For the teacher, it is useful to help students build a foundation from several small pieces of knowledge. For the student, it is useful to have a bunch of small victories through building tiny pieces of the puzzle that they can later use in building bigger solutions.

## Teaching Python for Data Science

In order to bring the above concepts together, I’ll bring you through a brief example of my experience co-learning and then teaching Python in the domain of data analysis and data science.

In 2017, I founded my second data science group. Previously, I had mainly organized speakers as my first data science group was in a bigger metropolitan area where there were already established data professionals who wanted to use the group for professionalization and networking. However, this new group was eager to learn new skills and to build community through a co-learning education process. I decided to build my first Python for data science course as an immersion course. This was very much a co-learning approach where there are not direct leaders: just a community of like-minded folks reading the same books and going through the same exercises that they would talk about on Slack and in person once a month.

The turnout for the first event was great. The place was packed despite the bad weather which is always a good sign. However, once they learned about the format, the numbers of attendees dropped off drastically. Over the 5 months of this course, it went from about 30 people interested in learning to only 3 who completed projects (one of which being me). So I immediately asked for feedback from those who attended and the larger community. It turned out that most people just wanted direct instruction with usable examples and well-organized code snippets they could use in their work. This was my eureka moment.

I realized that I hadn’t done any of the things I have mentioned above. I failed to know the people interested in the course and I failed to make it relevant to them. For those who stuck around, I failed to adjust material and format to ensure they stayed interested in not just learning Python, but also in building a bigger, local Python community.

So I rebuilt my course from the ground up. I made it more modular and left space for pauses, exercises, and demos. I make sure I’m constantly asking for feedback and that if they have issues or questions, that I’m adding those into the course material for our benefit. Once I was able to do these things, I found that more students stuck around for the course and that feedback was much better. Upon completion of my second iteration of teaching/co-learning, I already have requests for additional topics and for another run of the introductory materials.

I still have a long way to go as a teacher and as a Pythonista, but I found that bringing my strengths and weaknesses to the table helps everyone so long as I maintain my passion for the subject and help newcomers foster their own passions and motivations for learning and growing their skills.

## Credits

When building course materials for teaching Python, I drew on many sources. Many thanks to Wes McKinney, Jake VanderPlas, Dan Bader, Mike McKerns, Dan Dye, John Zelle, David Beazley, and many other people who made learning and using Python a blast!  
