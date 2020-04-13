---
title:  Latex: commands, declarations, groups, environments
---

Just like all the PhD students I know, I've beeen using Latex as a a collection of boilerplate code combined with frantic googling. At some point recently I got really stuck and went to the documentation to try and understand some of the grammar. I feel like it helped me set straight why sometimes things don't work when I could have sworn they should. And it made googling my problems a bit easier. These are the notes I tool when straying into LaTeX land.


## Control Sequences
The words *"command"* and *"control sequence"* are synonymous. In TeX the author, Donald Knuth called them control sequence in reference to cotrol characters, which existed before, in other comunication protocols. In LaTeX, later on the same concept is called a command. They are non-printing parts of the LaTeX code that define things about how the mark up will work.

Commands (or control sequences) are preceded by a backslash. They can be

1. special characters. They are for printing symbols that are normally reserved for commands.

    **Example:**
        
    ```
    \# \$ \% \^{} \& \_ \{ \} \~{} \textbackslash
    ```

2. of the format `\function[optional parameters]{non-optional parameters}`. Typcially, the command will apply to the parameters given and not produce side effects.
    
    **Example:**
    ```
    \textbf{this text is bold}
    ```

3. declarations. These are commands that don't take parameters. It directly affects the text and that effect  will  continue  until  another  declaration or until the environment or group ends (for group and environment see below!).

    **Example:**
    ```
    \large
    ```

    Declarations eat all white space that follows them. But, you can always put the (empty) curly braces, even if the command does not need them like `large{}`.

    A thing that I find hard to remember is which command is a declaration (i.e. changes everything that comes after forever) like `\large` and which take the text as an input and only format that, like `\textbf{my fat text}`. This can get particularly nasty because curly braces do double duty: they indicate parameters in a command but they also define **groups** (see below).

    ![]({{site.blog_url}}/resources/images/blog5/braces_a.png){:height="200px" class="shadow" class="center" }
   
    The braces put the word "big" in its own group (that has no special properties). The size change applies to all following words.

    ![]({{site.blog_url}}/resources/images/blog5/braces_b.png){:height="200px" class="shadow" class="center" }
    
    The braces after the declarations are empty and all they do is stop it from gobbling subsequent whitespace. The size change applies to all following words.

    ![]({{site.blog_url}}/resources/images/blog5/braces_c.png){:height="200px" class="shadow" class="center" }
    
    The braces contain the size change to a group which contains only the word "huge". The size change only applies within the group.

    As you can see from the first example, if I'm not sure if `\huge`is a declaration or a parametrized command, I can get unexpected results, because the curly braces after a command keyword can have different meanings based on whether the command expects parameters or not. In the first example they are interpreted as the start of a group.

    But wait- its gets weirder: if you have a command like `\textbf` that expects a parameter and don't give it one, it affects the first letter that comes after it.
    
    ![]({{site.blog_url}}/resources/images/blog5/braces_d.png){:height="200px" class="shadow" class="center" }

## Groups and environments
**Groups** are a way of limiting the reach of certain commands. They are evoked simply by curly braces. Environments are also a type of group. You can hierarchically define groups as long as you can keep track of all your braces. Settings/declarations from an group higher up the hierarchy are carried over, into the children. changes in the children do not affect the parent groups.

**Environments** are framed by `\begin` and `\end` commands. They define how the contents is formatted. A list of all prespecified environments can be found [here](https://latex.wikia.org/wiki/List_of_LaTeX_environments).


## Your own function or environment
To make new commands use `\newcommand{name}[num]{definition}`.

**example:**

        `\newcommand{\enquote}[1]{``#1''} %put in the right quotes`

![]({{site.blog_url}}/resources/images/blog5/enquote.png){:height="200px" class="shadow" class="center" }

To make new environments use `\newenvironment{name}[num]{before}{after}`.
**example:** drawing a box around the contents

        \newenvironment{boxed} % this is the name. It does not take any input parameters
            { % this is the "before"
            \begin{center}
            \begin{tabular}{|p{0.9\textwidth}|}
            \hline\\
            }
            { % this is the "after"
            \\\\\hline
            \end{tabular}
            \end{center}
            }

![]({{site.blog_url}}/resources/images/blog5/box.png){:height="300px" class="shadow" class="center" }

## Centering things 3 ways
So considering all of the above we can achieve the same thing in a variety of ways:

1. Command with parameters

        \centerline{This line will be centered}

2. This is a declaration which is valid until the end of the group

        {\centering This line will be centered}

3. This is an Environment (inbuilt)

        \begin{center}
            This is line one. \\This is line two. \\This is line three.
        \end{center}

I have found my problems usually happen when I mix up the keywords and, for example, attempt to use the declaration syntax as a parametrized command which leads to my curly braces being interpreted as a group.


## Extra useful tidbits:
- \emph will emphasize intelligently depending on context. Usually that means cursive, but when everything around it is already cursive it will emphasize it a different way.
- the todonotes package is the best thing ever!
- the command `\interlinepenalty=10000` in thebibliography section stops citations from breaking over multiple pages. Works like a charm!
