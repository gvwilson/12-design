Most people can lift one kilogram, but would struggle to lift one
hundred, and could not lift a thousand without planning and support.
Similarly, most researchers can write a few lines of Python, R, or
MATLAB to create a plot, but would struggle to create a program that
was a few hundred lines long, and wouldn't know where to start
building an application with many thousands of lines spread across
dozens of modules or packages.  While you may not (yet) write software
at that scale, knowing how to design in the large helps ensure that
you are pointed in the right direction.

# Tip 1: design after the fact.

Researchers often don't know what their code should do tomorrow until
they've seen today's results. Lots of up-front design is therefore
often unprofitable, but this isn't a license to create a tangled mess:
we can and should make it look as though we knew where we were going
so that the next person will be able to understand our work
[@Parnas1986].

*Refactoring* is the process of rewriting code without changing its
behavior. [@Fowler2018] describes common refactorings, such as
"extract function" (i.e., move some code out of the function its in
and put it in a new function that can be called separately). Just as
the tidying steps in a data pipeline either convert messy data to a
tidy layout or move the data from one tidy layout to another
[@Wickham2017], most refactoring operations move code toward or
between well-defined patterns [@Kerievsky2004].

# Tip 2: design for people's cognitive capacity.

[@Miller1956] estimated that the average person could hold $7{\pm}2$
items in short-term memory at once, and more recent estimates put its
capacity closer to $4{\pm}1$.  The second tip for software design is
therefore to minimiz the number of "things" someone has to remember at
any time to understand the code they're looking at. Breaking code into
many short functions, each of which takes only a few parameters, and
defining default values for most of those parameters, both aid this.

Another approach is to create families of functions that take similar
inputs and outputs and combine them using *pipes* so that readers only
have to "carry forward" one piece of information at a time. This model
was one of Unix's key contributions to computing [@Kernighan2019], and
has been adopted by other programming systems like the "tidyverse"
family of packages in R [@Wickham2017].

# Tip 3: design in coherent levels.

Every function should implement a single logical operation. The
easiest way to check if it does so is to read it aloud and ask if the
steps are at the same conceptual level. For example, if I read this
Python function aloud:

    def main():
        config = buildConfiguration(sys.argv)
        state = initializeState(config)
        while config.currentTime < config.haltTime:
            updateState(config, state)
        report(config, state)

the comparison in the `while` loop feels like a lower level of detail
than the function calls. Replacing it with another function called
something like `stillEvolving` feels more uniform:

    def main():
        config = buildConfiguration(sys.argv)
        state = initializeState(config)
        while stillEvolving(config, state):
            updateState(config, state)
        report(config, state)

# Tip 4: design for evolution.

The replacement shown above also makes future evolution easier. If we
decide the simulation should run until a specified time *or* until its
state has stabilized, we can make that change in the `stillEvolving`
function without modifying anything else.

This example shows that good design enables independent evolution of
parts, so that a fix *here* doesn't require changes *there*.
Programmers achieve this by separating *interface* (what a piece of
software can do) from *implementation* (how it achieves that). Doing
this enables construction of components that can be plugged together
like USB devices. Many advanced features of programming languages
exist to support or enforce this, such as being able to derive classes
in object-oriented languages like Python, or creating generic
functions that do the same logical thing in different ways for
different types of data in languages like R and Julia.

# Tip 5: group related information together.

If several things are closely related or frequently occur together,
our brains combine them into a "chunk" that only takes up one slot in
short-term memory [@Thalmann2019]. We can aid this by combining
related values into data structures. For example, instead of managing
the XYZ coordinates of points separately like this:

    def enclose(x0, y0, z0, x1, y1, z1, nearness):
        ...

we can store points' coordinates in structures pass those around

    def enclose(p0, p1, nearness):
        ...

Grouping related information also aids code evolution.  For example,
if we decide to use radial coordinates instead of Cartesian
coordinates, we can change how points are represented without changing
the functions that pass them around.

# Tip 6: use common patterns.

Some chunks appear so often that we call them "patterns" and give them
names.  Good programmers use these *design patterns* to reduce the
amount of thinking they have to do and because these patterns have
proven to be useful in the past.  Patterns can be found at all scales
of programming: "most valuable" variables [@Byckling2005],
doubly-nested loops to process the elements of two-dimensional arrays,
and the filter-group-summarize pattern common in data analysis are
just three examples.  Learning these patterns helps make someone a
better programmer [@Tichy2010].

# Tip 7: design for delivery.

Developer operations (DevOps) is a shorthand term for automating
compilation, packaging, distribution, deployment, and monitoring so
that programmers can focus on other problems [@Kim2016;
@Forsgren2018].  Investment in automation pays off many times over,
but only if you design things so that they can be
automated. [@Taschuk2017] lays out some rules for doing this, and a
few others include:

1.  Use the same tools to build packages as everyone else who uses your
    language, e.g., `pip` for Python or `devtools` for R.

2.  Organize your source files in the way your build system expects.

3.  Use a logging library rather than `print` commands to report what
    your program is doing and any errors it has encountered.

# Tip 8: design for testability.

Research software is notoriously difficult to test [@Hook2009;
@Kanewala2014], in part because its developers often don't know what
output it's supposed to produce. (As a frustrated chemist once said to
the author, "If I knew what the right answer was, I'd have published
it already.") It *is* possible to test the parts of an application
that parse input files, clean up messy data, etc., and doing this
makes it easier to refactor code [@Feathers2004], but we can only
create tests if we design the software in testable pieces. We can
check how well we've done this by asking:

-   How easy is it to create the *fixture* that a test runs on?

-   How easy is it to invoke just the behavior we want?

-   How easy is it to check the result?

-   How easy is it to figure out what "right" is?

-   How easy is it to delete the feature?

# Tip 9: design as if code was data.

Code is just another kind of data: programs are just text files, and
once a program is loaded into memory it is just another data structure
whose bytes are instructions rather than characters, numbers, or
pixels. The first fact allows us to use tools that process it in many
ways, provided the code is designed with these tools in mind. Examples
include:

-   style-checking tools that check that the layout, variable names, and
    other properties conform to coding standards;

-   documentation tools that extract specially-formatted comments and
    create cross-referenced manual pages; and

-   indexing and navigation tools that enable us to jump directly to the
    definition of a function or variable.

The second insight---the fact that a function in memory is just
another kind of data---is the basis of many techniques for shortening
and reusing code:

-   Passing functions as arguments to other functions so that common
    operations only have to be written once.

-   Storing functions in data structures so that new operations can be
    added to a program without changing any of the pre-existing code.

-   Loading modules based on configuration parameters (as in the earlier
    example of getting species information).

Many features in modern languages, such as lazy evaluation in R or
decorators in Python, leverage this insight, and taking advantage of it
can make code much smaller and easier to understand---but only if you
remember that what is powerful in the hands of experts is spooky
action-at-a-distance for novices.

# Tip 10: design graphically.

Many programmers sketch when they're designing. These sketches are
usually not meant as blueprints: instead, they help people
*externalize cognition*, i.e., get their thoughts out where they can
see them [@Cherubini2007; @Petre2016].  Among the drawings programmers
often find helpful are:

-   flowcharts showing possible paths through a piece of code;

-   entity-relationship diagrams showing how database tables relate to
    one another; and

-   concept maps showing how the designer thinks about the overall
    problem.

In most cases, the act of drawing (with or without a collaborator) is
more important than the drawing itself.

# Tip 11: design with everyone in mind.

Fairness, privacy, and security cannot be sprinkled onto software
after the fact. Designers should therefore follow the Principle of
Least Privilege: every component in the system should require as few
permissions as possible for as short as possible.

But there is much more to safety-conscious design than just data
protection. For example, if an application requires users to change
their password every few weeks, *security fatigue* will soon set in and
people will choose less and less secure passwords [@Smalls2021].
Similarly, programs should not email files to people: doing that trains
them to open attachments, which is a common channel for attacks.

Accessibility also can't be sprinkled onto software after the fact.
Close your eyes and try to navigate your institution's website. Now
imagine having to do that all day, every day. Imagine trying to use a
computer when your hands are crippled by arthritis. Better yet, don't
imagine it: have one of your teammates tape some popsicle sticks to your
fingers so you can't bend them and see what it's like to reply to an
email.

A good short guide for accessible design is a set of posters from the
UK Home Office [@UKHO]. Each poster in this series lays out a few
simple do's and don'ts that will help make your software accessible to
people who are neurodivergent, use screen readers, are dyslexic, have
physical or motor challenges, or are hard of hearing.

# Tip 12: design for contribution.

Study after study has shown that diversity improves outcomes in fields
from business to healthcare [@Gompers2018; @Gomez2019]. Good design
therefore makes it easier for people who aren't already immersed in
your project to figure out where and how they can contribute to it
[@Sholler2019]:

-   Software licensing is a design issue, since a program can't use
    libraries whose licenses are incompatible with its own. Many
    designers therefore prefer permissive licenses like the MIT License
    to maximize the number of people who will be able to take advantage
    of their work.

-   Applications that support plug-ins are often easier for newcomers to
    contribute to, as are libraries with strong and consistent
    conventions for passing data (like Unix command-line tools or R's
    tidyverse functions).
