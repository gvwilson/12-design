# Twelve Software Design Tips for Data Scientists

Greg Wilson | http://third-bit.com

## Introduction

Most people can lift one kilogram, but would struggle to lift one hundred, and
could not lift a thousand without planning and support.  Similarly, most
researchers can write a few lines of Python, R, or MATLAB to create a plot, but
would struggle to create a program that was a few hundred lines long, and
wouldn't know where to start building an application with many thousands of
lines spread across dozens of modules or packages.

The core challenge is that *programming in the large* is qualitatively different
from *programming in the small*.  as the number of pieces in a program grows,
the number of possible interactions between those pieces grows much more quickly
because <em>N</em> components can be paired in <em>N<sup>2</sup></em> ways.
Programmers who don't manage this complexity quickly wind up with software that
behaves in unexpected (and usually unfortunate) ways and that cannot be modified
without heroic effort.

Most people don't have to write software at that scale, thanks in large part to
an ever-expanding ecosystem of open source software, but someone has to.  Since
many big programs begin life as small scripts, knowing how to design in the
large helps ensure that what's done in the small is pointed in the right
direction.

This paper describes a dozen rules that can help data scientists design
large programs. These rules are taken from published sources, the
author's personal experience, and discussions over thirty-five years
with the creators of widely-used libraries and applications.

### Acknowledgments

The author is grateful to Sam Albers, Rebecca Barter, Neil Brown,
Matthias Bussonnier, Daniel Chen, Ildi Czeller, Bradford Dykes, Damien
Irving, Mandip Mistry, and Donny Winston for feedback on early
versions of this paper.

## Rule 1: design after the fact

When we are doing research, we often don't know what the code should do
tomorrow until we've seen today's results. It's therefore often
pointless to invest as much in up-front planning for our software as we
would in drawing up blueprints for a building. But this isn't a license
to create a tangled mess: we can and should make it look as though we
knew what we were going to do in advance so that the next person who has
to read our code will be able to understand it [Parnas1986].

*Refactoring* is the process of reorganizing or rewriting code without
changing its externally-visible behavior. [Fowler2018] describes common
refactorings, such as "extract function" (i.e., move some code out of
the function its in and put it in a new function so that it can be
called separately) and "combine parameters into object" (which we we
will revisit in Rule 5). Just as the tidying steps in a data pipeline
either convert messy data to a tidy layout or move the data from one
tidy layout to another [Wickham2017], most refactoring operations move
code toward or between well-defined patterns [Kerievsky2004].

> Many designers explain the design of their software by recapitulating
> its history [Brown2011; Brown2012]. This is sometimes called
> *challenge and response*: the only way to understand why something
> works the way it does is to understand the problems that existed at
> the time it was written and the tools that were available then.
> Neither programming languages nor existing graphical notations (Rule 10)
> are particularly good at capturing this, and while many tools have
> been written for tracking and managing requirements, none have worked
> well enough that the average programmer would voluntarily adopt them.

## Rule 2: design for people's cognitive capacity

Just as computers have hard drives and RAM, human brains have long-term
and short-term memory [Hermans2021]. We only have conscious access to
short-term memory, and it is much smaller than most people realize:
early estimates were that the average person could hold 7±2 items
in short-term memory at once [Miller1956], and more recent estimates
put its capacity closer to 4±1.

The second rule of software design is therefore to ensure that the
number of "things" someone has to remember at any time in order to
understand the code they're looking at fits in short-term memory. For
example, if a function takes 37 parameters then the odds are slim that
someone will remember them all and put them in the right order without a
lot of trial and error. Breaking the function into pieces, each of which
takes only a handful parameters, reduce the *cognitive load*. Similarly,
defining default values for most or all of the parameters will allow
people to ignore the details most of the time.

A complementary approach is to create families of functions that all
take similar inputs and outputs and then combine them using *pipes*. The
pipe operator is one of Unix's key contributions to computing
[Kernighan2019], and the idea has been adopted by many other
programming systems, including the "tidyverse" family of packages in R
[Wickham2017]. Where a mathematician would write <em>f(g(h(x)))</em>, a
programmer using pipes would write <em>h(x)|g|f</em> so that the functions'
names appear in the order in which they are applied. This may seem like
a small change, but aligning the reading order with execution order
makes the overall effect easier to understand because the reader only
has to "carry forward" one piece of information at a time.

## Rule 3: design in coherent levels

Another rule for designing functions is that each function should be
short, shallow, and single-purpose, i.e., it should implement a single
mental operation so that it only takes up a single slot in short-term
memory. The easiest way to check if this rule is being followed is to
read the function aloud and ask whether all the steps are at the same
conceptual level. For example, if I read this Python function aloud:

     def main():
         config = buildConfiguration(sys.argv)
         state = initializeState(config)
         while config.currentTime < config.haltTime:
             updateState(config, state)
         report(config, state)

the highlighted comparison of the current time to the halting time in
the `while` loop feels like it's at a lower level of detail than the
function calls.  I would probably replace the comparison with a
function called something like `stillEvolving`:

     def main():
         config = buildConfiguration(sys.argv)
         state = initializeState(config)
         while stillEvolving(config, state):
             updateState(config, state)
         report(config, state)

This rewrite saves the reader from having to jump between two different
levels of detail while they're trying to figure out what this function
does.

## Rule 4: design for evolution

The change shown above also makes future evolution easier. If we decide
that the simulation should run until a specified time *or* until its
state has stabilized, we can make that change in the `stillEvolving`
function without modifying anything else.

Hiding details like this is another general rule of software design.
Software changes over time because our problems change, our environments
change, and because our skills improve. A good design makes independent
evolution of parts easier: basically, a fix *here* shouldn't require
changes *there*. More realistically, a change in one place should only
require a small number of changes in a few predictable places.

Software designers achieve this through *information hiding* and *loose
coupling*:

-   *Information hiding* means putting the data that part of a program
    needs to do its job in one place, and only accessing it through a
    small set of functions. Doing this makes no difference to the
    computer—by the time it runs the program, it's all just a bunch of
    instructions—but people need to understand the program, not
    execute it.

-   Doing this allows *loose coupling*, i.e., those pieces of software
    can be combined in many different ways because none of them depends
    on the internals of any of the others.

Programmers usually implement these principles by separating *interface*
from *implementation*. The interface specifies what a piece of software
can do; its implementation is how it achieves that, and no other piece
of software should be tied to its implementation details. The goal is to
enable the construction of software components that can be mixed and
matched in the way that USB interfaces allow pieces of hardware to be
combined. Many of the more advanced features of programming languages
exist to support or enforce this, such as being able to derive classes
from one another in object-oriented languages like Python, or creating
generic functions that do the same logical thing in different ways for
different types of data in languages like R and Julia.

*Design by contract* is one way to enforce this idea [Meyer1994]. Any
function or method can be characterized by the *pre-conditions* that
must be true of its inputs in order for it to run and the
*post-conditions* that it guarantees will be true of its output. For
example, a function's pre-conditions might be that its input must be an
array of numbers in sorted order, and its post-condition might be that
the value it returns is one of those elements.

If design by contract is followed, then a replacement for this function
can weaken the pre-conditions so that it handles a larger set of inputs
and/or strengthen the post-conditions so that it produces a narrower set
of outputs, but not vice versa. The first rule ensures that the new
function will be able to handle everything that the old one could; the
second ensures that anything using the old function's output will still
be able to handle everything the new one produces. Continuing with the
previous example, a new function's pre-conditions could be that it takes
any array of numbers, not just one that is sorted, and that it produces
the largest element in the array.

## Rule 5: group related information together

If several things are closely related or frequently occur together,
our brains combine them into a "chunk" that only takes up one slot in
short-term memory [Thalmann2019]. We can aid this by combining related
values into data structures. For example, instead of storing the X, Y,
and Z coordinates of points separately like this:

    def enclose(x0, y0, z0, x1, y1, z1, nearness):
        ...

we can store each point's coordinates in a structure and write our code
like this:

    def enclose(p0, p1, nearness):
        ...

This allows us to thing about each point as one ``thing''.  Where we
need the individual coordinates, we can refer to them as `p0.X`,
`p0.Y`, and so on.

Grouping related information together also aids code evolution
(Rule 4).  If, for example, we decide to use radial coordinates
instead of Cartesian coordinates, we can change how points are
represented without changing most of the functions that pass them
around.

## Rule 6: use common patterns

Some chunks appear so often that we call them "patterns" and give them
names. For example, parasitism and symbiosis take many forms, and it's
impossible to draw a precise boundary between them, but they are
powerful ideas that help us make sense of the world.

Good programmers use *design patterns* to structure their code, both to reduce
the amount of thinking they have to do and because these patterns have proven to
be useful in the past. Learning patterns helps make someone a better programmer,
i.e., there is causation, not just correlation [Tichy2010].  Conforming to
widely-understood patterns also makes code more comprehensible, just as dividing
this paper into sections and a bibliography that are consistent with what you've
read before makes it easier to read.

Patterns can be found at all scales of programming, from "most valuable"
variables [Byckling2005] and doubly-nested loops to process the
elements of two-dimensional arrays through the filter-group-summarize
pattern that is common in data analysis, to the communicating
microservices of today's online applications. But design patterns can be
a mixed blessing. First, our brains try so hard to match inputs to
patterns that they will sometimes misclassify things. For example, it
once took me the better part of an hour to spot the error in this code:

     for (i=0; i<a.width; i++) {
         for (j=0; i<a.height; j++) {
             a[i][j] = cos(abs(a[i][j]) - lemaitre(b_norm, a[j][i]))
         }
     }

The problem is in the second line, which mistakenly uses the variable
`i` where it should use the variable `j`. I couldn't find the mistake
because the code was so close to what it should have been that my brain
"corrected" what my eyes were seeing, and because I assumed that since
the third line was the most complicated, the error had to be there.

> In the same way that you shouldn't write a math problem with a
> function `x` of variable `g`, you should use the same naming
> conventions as your peers to make your code easier to read. It doesn't
> matter what these are, any more than it matters whether you spell
> "color" the correct way or the British way; what matters is the
> predictability that comes from consistency.

This example illustrates a corollary to Rule 2: we should write programs
that maximize the ratio of unique code to boilerplate. In most modern
languages, the five lines shown above can be written as:

    a = cos(abs(a) - lemaitre(b_norm, a.transpose()))

This version is probably as efficient as the first, but it is much
easier to read and therefore much less likely to mislead.

## Rule 7: design for delivery

Developer operations (DevOps) has become a buzzword in the last few
years. Like "data science" or "computational thinking", the term is
popular because people can use it to mean whatever they want, but the
core idea is a good one: automate compilation, packaging, distribution,
deployment, and monitoring by writing software that operates on or
monitors other software [Kim2016; Forsgren2018].

Investment in automation pays off many times over, but only if you
design things so that they can be automated. [Taschuk2017] lays out
some rules for doing this, and a few others include:

1.  Use the same tools to build packages as everyone else who uses
    your language, e.g., `pip` for Python or `devtools` for R.

2.  Organize your source files in the way your build system expects so
    that those tools can do their job.

3.  Handle errors rather than catching and discarding them
    [Nakshatri2016].

4.  Use a logging library rather than `print` commands to report what
    your program is doing and any errors it has encountered. Logging
    libraries allow you to enable and disable certain categories of
    messages selectively. This is very useful during development, but
    even more so in production, since it lets whoever is using your
    software turn reporting on without having to rebuild anything.

## Rule 8: design for testability

Research software is notoriously difficult to test [Hook2009,Kanewala2014], in
part because its developers often don't know precisely what output the code is
supposed to produce.  (As a frustrated chemist once said to the author, "If I
knew what the right answer was, I'd have published it already.")  It *is*
possible to test the "other 90%" of a typical data science application—the parts
that parse input files, clean up messy data, and so on, but only if those parts
are designed with testing in mind.

*Legacy code* is software that we're afraid to try to modify because
it's hard to understand and things will break unexpectedly. A
comprehensive set of tests makes us less afraid [Feathers2004], but we
can only create those tests if we design the software in testable
pieces. We can check how well we've done this by asking:

-   How easy is it to create a *fixture* (i.e., the input that a test
    runs on)?

-   How easy is it to invoke just the behavior we want?

-   How easy is it to check the result?

-   How easy is it to figure out what "right" is?

-   How easy is it to delete the feature?

For example, suppose that our program uses species data stored in a
large database. Since our tests only rely on a handful of facts about
half a dozen species, we can:

1.  Write a function called `getSpeciesInfo(speciesId)` that gets
    information about a species given its unique ID and put that
    function in a module called `production`.

2.  Create a second module called `testing` that contains a function
    with the same name that takes the same species ID and returns a
    value from a hard-coded lookup table. This table only includes the
    facts we need about the species we use in testing.

3.  Use a run-time flag to control which of these two modules the
    program actually loads when it runs.

Once again, funnelling all requests for species information through one
function doesn't just make testing easier today. It also ensures that if
we want to change how information is looked up in future we are certain
that we'll only have to modify one small piece of code.

## Rule 9: design as if code was data

The insight on which all modern computing is based is that code is just
another kind of data. Programs are just text files, and once a program
is loaded into memory it is just another data structure: instead of
interpreting its bytes as characters or pixels, we interpret them as
instructions, but they're still just bytes.

The fact that source code is stored in text files allows us to build
tools that process it, providing the software is designed with those
tools in mind. Examples include:

-   style-checking tools that check that the layout, variable names, and
    other properties conform to coding standards;

-   documentation tools that extract specially-formatted comments and
    create cross-referenced manual pages; and

-   indexing and navigation tools that enable us to jump directly to the
    definition of a function or variable.

It is the second insight—the fact that a function in memory is just
another kind of data—that has the most impact on design. Just a few of
the ways it shows up are:

-   Passing functions as arguments to other functions so that common
    operations only have to be written once.

-   Storing functions in data structures so that new operations can be
    added to a program without changing any of the pre-existing code.

-   Loading modules based on configuration parameters (as in the earlier
    example of getting species information).

For example, if we want to count the number of positive values in an array, we
can write:

    def count_positive(array):
        number = 0
        for value in array:
            if value >= 0:
                number = number + 1
        return number

If we want to count the number that are negative, we could write a
`count_negative` function that differed by only one character (replacing
the `>` with `<`). Alternatively, we could write a generic function like
this:

    def count_interesting(array, test):
        number = 0
        for value in array:
            if test(value):
                number = number + 1
        return number

then put each test in a function like this:

    def is_positive(value):
        return value >= 0

and then write something like:

    count_interesting(pressures, is_positive)

Many features in modern languages, such as lazy evaluation in R or
decorators in Python, leverage this insight, and taking advantage of it
can make code much smaller and easier to understand—but only if you
remember that what is powerful in the hands of experts is spooky
action-at-a-distance for novices.

> The best balance between abstraction and detail depends on how much people
> already know. For a novice, too many low-level details obscure the meaning,
> while too much abstraction makes it impossible to figure out what's actually
> going on (<a href="#comprehension">Figure 1</a>). As that person gains
> experience they become better able to synthesize meaning from detail and
> translate generalities into specifics, but their optimum balance also shifts.
> Software that is easiest for them to understand may not be optimal for someone
> else.

<figure id="comprehension">
<img src="./comprehension.png" alt="Comprehension curves" />
<figcaption>Figure 1: Comprehension curves</figcaption>
</figure>

## Rule 10: design graphically

Many formal graphical notations for software have been designed over the
years. The most famous is the Unified Modeling Language (UML), but in
practice it is taught more often than it is used, and when it *is* used,
it is usually not in the ways its creators intended [Petre2013].

However, many programmers do sketch when they're designing, and these
sketches do help them design. These sketches are usually not meant as
blueprints: instead, they help people *externalize cognition*, i.e., get
their thoughts out where they can see them [Cherubini2007; Petre2016].
Among the drawings that working programmers often find helpful are:

-   flowcharts, which are unfairly maligned [Scanlan1989]
    (<a href="#flowchart">Figure 2</a>);

-   entity-relationship diagrams showing how database tables relate to
    one another (<a href="#er-diagram">Figure 3</a>);

-   concept maps showing how the designer thinks about the overall problem
    (<a href="#concept-map">Figure 4</a>); and

-   many others, such as dataflow diagrams, system architecture diagrams
    showing the major components of an application, and use case maps
    that show how activity flows through an architecture [Reekie2006].

<figure id="flowchart">
<img src="./flowchart.png" alt="Flowchart" />
<figcaption>Figure 2: Flowchart</figcaption>
</figure>

<figure id="er-diagram">
<img src="./er-diagram.png" alt="Entity-relationship diagram" />
<figcaption>Figure 3: Entity-relationship diagram</figcaption>
</figure>

<figure id="concept-map">
<img src="./concept-map.png" alt="Concept map" />
<figcaption>Figure 4: Concept map</figcaption>
</figure>

## Rule 11: design with everyone in mind

If the last few years have taught us anything about software, it's that
fairness, privacy, and security cannot be sprinkled on after the fact.
For example, if a program is initially designed in a way that allows
every user to see everyone else's data, adding privacy controls later
will be expensive and almost certainly buggy.

But there is much more to safety-conscious design than just data protection.
For example, if an application requires users to change their password
every few weeks, *security fatigue* will soon set in and people will
choose less and less secure passwords [Smalls2021]. Similarly, programs
should not email files to people: doing that trains them to open
attachments, which is a common channel for attacks. And every
application should allow people to erase data, which means its data
structures and database tables have to be designed to allow for actual
erasure, not just "mark as inactive".

Accessibility also can't be sprinkled onto software after the fact.
Close your eyes and try to navigate your institution's website. Now
imagine having to do that all day, every day. Imagine trying to use a
computer when your hands are crippled by arthritis. Better yet, don't
imagine it: have one of your teammates tape some popsicle sticks to your
fingers so you can't bend them and see what it's like to reply to an
email.

Making software accessible doesn't just help people with obvious
disabilities: the population is aging, and everything you do to help
people who are deaf also helps people who are gradually losing their
hearing [Johnson2017]. A good short guide for accessible design is a
set of posters from the UK Home Office [UKHO]. Each poster in this
series lays out a few simple do's and don'ts that will help make your
software accessible to people who are neurodivergent, use screen
readers, are dyslexic, have physical or motor challenges, or are hard of
hearing.

## Rule 12: design for contribution

Study after study has shown that diversity improves outcomes in fields
from business to healthcare because many perspectives make for fewer
errors [Gompers2018; Gomez2019]. Good design makes it easier for
people who aren't already immersed in your project to figure out where
and how they can contribute to it [Sholler2019], but just as good
programmers consider things like packaging and deployment in their
designs, so too do they think about the factors that influence
contribution:

-   Software licensing is a design issue, since a program can't use
    libraries whose licenses are incompatible with its own. Many
    designers therefore prefer permissive licenses like the MIT License
    to maximize the number of people who will be able to take advantage
    of their work.

-   Applications that support plug-ins—i.e., that allow people to
    write small modules that the main program can load and use—are
    often easier for newcomers to contribute to. Similarly, libraries
    with strong and consistent conventions for passing data (like Unix
    command-line tools or R's tidyverse functions) enable people to
    start with small contributions.

## Conclusion

<a href="#bicycle">Figure 6</a> is a De Rosa SK Pininfarina bicycle.
It is not a work of art, but it *was* carefully designed, and as a
result it can be analyzed and appreciated esthetically.  Rules like
the ones in this paper and in longer works like [Budgen2020] may one
day help us create data analyses that also deserve to be called
"beautiful".

<figure id="bicycle">
<img src="./derosa.jpg" alt="De Rosa bicycle" />
<figcaption>Figure 6: De Rosa SK Pininfarina bicycle (image from https://derosanorthamerica.com/)</figcaption>
</figure>

## Bibliography

[Brown2011] Brown A, Wilson G, editors. *The Architecture of Open Source Applications: Elegance, Evolution, and a Few Fearless Hacks*. Lulu; 2011. Available from: <https://aosabook.org>.

[Brown2012] Brown A, Wilson G, editors. *The Architecture of Open Source Applications: Structure, Scale, and a Few More Fearless Hacks*. Lulu; 2012. Available from: <https://aosabook.org>.

[Budgen2020] Budgen D.: *Software Design* (3rd ed.). Routledge; 2020, 978-1138196612.

[Byckling2005] Byckling P, Gerdt P, Sajaniemi J. "Roles of Variables in Object-Oriented Programming". In: *Proc. OOPSLA'05*. ACM; 2005.

[Cherubini2007] Cherubini M, Venolia G, DeLine R, Ko AJ. "Let's go to the whiteboard: how and why software developers use drawings." In: *Proc. CHI'07*. ACM; 2007.

[Feathers2004] Feathers MC. *Working Effectively with Legacy Code*. Prentice-Hall; 2004.

[Forsgren2018] Forsgren N, Humble J, Kim G. *Accelerate: The Science of DevOps*. Revolution Press; 2017.

[Fowler2018] Fowler M. *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley Professional; 2018.

[Gomez2019] Gomez LE, Bernet P. "Diversity improves performance and outcomes." *Journal of the National Medical Association*. 2019;111(4):383–392. doi:10.1016/j.jnma.2019.01.006.

[Gompers2018] Gompers P, Kovvali S. "The Other Diversity Dividend." *Harvard Business Review*. 2018;96(4):72–77.

[Hermans2021] Hermans F. *The Programmers Brain: What every programmer needs to know about cognition*. Manning; 2021.

[Hook2009] Hook D, Kelly D. "Testing for trustworthiness in scientific software." In *Proc. 2009 ICSE Workshop on Software Engineering for Computational Science and Engineering, 2009, doi:10.1109/secse.2009.5069163.

[Johnson2017] Johnson J, Finn K. *Designing User Interfaces for an Aging Population: Towards Universal Design*. Morgan Kaufmann; 2017.

[Kanewala2014] Kanewala U, Bieman JM. "Testing scientific software: A systematic literature review." *Information and Software Technology*, 56(10), 2014, doi:10.1016/j.infsof.2014.05.006.

[Kerievsky2004] Kerievsky J. *Refactoring to Patterns*. Addison-Wesley Professional; 2004.

[Kernighan2019] Kernighan BW. *UNIX: A History and a Memoir*. Independently published; 2019.

[Kim2016] Kim G, Debois P, Willis J, Humble J. *The DevOps Handbook*. Revolution Press; 2016.

[Meyer1994] Meyer B. *Object-Oriented Software Construction*. Prentice-Hall; 1994.

[Miller1956] Miller GA. "The Magical Number Seven, Plus or Minus Two: Some Limits on Our Capacity for Processing Information." *Psychological Review*. 1956;63(2):81–97. doi:10.1037/h0043158.

[Nakshatri2016] Nakshatri S, Hegde M, Thandra S. "Analysis of exception handling patterns in Java projects." In: *Proc. MSR'13*. ACM; 2016.

[Parnas1986] Parnas DL, Clements PC. "A Rational Design Process: How and Why to Fake It". *Transactions on Software Engineering*. 1986;SE-12(2):251–257. doi:10.1109/tse.1986.6312940.

[Petre2013] Petre M. "UML in practice". In: *Proc. ICSE'13*; 2013. p. 722–731.

[Petre2016] Petre M, van der Hoek A. *Software Design Decoded: 66 Ways Experts Think*. MIT Press; 2016.

[Reekie2006] Reekie J, McAdam R. *A Software Architecture Primer*. Angophora Press; 2006.

[Scanlan1989] Scanlan DA. "Structured Flowcharts Outperform Pseudocode: An Experimental Comparison." *IEEE Software*. 1989;6(5):28–36. doi:10.1109/52.35587.

[Sholler2019] Sholler D, Steinmacher I, Ford D, Averick M, Hoye M, Wilson G. "Ten simple rules for helping newcomers become contributors to open projects." *PLoS Computational Biology*. 2019;15(9):e1007296. doi:10.1371/journal.pcbi.1007296.

[Smalls2021] Smalls D, Wilson G. "Ten quick tips for staying safe online." *PLoS Computational Biology*. 2021;17(3):e1008563. doi:10.1371/journal.pcbi.1008563.

[Taschuk2017] Taschuk M, Wilson G. "Ten Simple Rules for Making Research Software More Robust." *PLoS Computational Biology*. 2017;13(4). doi:10.1371/journal.pcbi.1005412.

[Thalmann2019] Thalmann M, Souza AS, Oberauer K.: "How does chunking help working memory?" *Journal of Experimental Psychology: Learning, Memory, and Cognition*, 45(1), 2019, doi:10.1037/xlm0000578.

[Tichy2010] Tichy W. "The Evidence for Design Patterns." In: Oram A, Wilson G, editors. *Making Software*. O'Reilly; 2010.

[UKHO] UK Home Office. "Designing for accessibility"; viewed August 2021. Available from: <https://ukhomeoffice.github.io/accessibility-posters/posters/accessibility-posters.pdf>.

[Wickham2017] Wickham H, Grolemund G. *R for Data Science: Import, Tidy, Transform, Visualize, and Model Data*. O'Reilly; 2017.
