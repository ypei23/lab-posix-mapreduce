

Recall that in last week's lab, we saw how to use python and SQL to perform the "count distinct" and "group by" queries.
We had two main takeaways:
1. Python's pandas library requires $\Omega(n)$ memory,
    and so it is not suitable for large datasets.
1. SQLite requires only $O(1)$ memory,
     and so is suitable for large datasets.
We will not see how to run those same queries in the shell.

## Part 0: Setup

Fork this repo.
$ git clone

## Part 1: Combining Python with the shell

The files `colors` contains a small dataset of different colors.
Examine it.
```
$ cat colors
```
In this section, we will write some short python code to perform out count distinct and count groupby queries.

### Part 1.a: Creating the files

Create a file `count_distinct.py` with the following python code.
```
#!/usr/bin/python3
import sys
unique_lines = set()
for line in sys.stdin:
    unique_lines.add(line[:-1])
print(len(unique_lines))
```
This program loops over the contents of stdin and adds each line to the `unique_lines` set.
The `set` container automatically removes duplicates,
and so the final `print` statement will print the total number of distinct lines.

Try it out with the following commands
```
$ chmod a+x count_distinct.py
$ cat colors | ./count_distinct.py
```

A similar python program can be used to perform the count group by operation.
All we have to do is replace the `set` with `Counter`.

Create a file `count_groupby.py` with the following code,
and give yourself execute permissions.
```
#!/usr/bin/python3
import sys
from collections import Counter
unique_lines = Counter()
for line in sys.stdin:
    unique_lines[line[:-1]] += 1
print(dict(unique_lines))
```
And test the program on the `colors` file.
```
$ cat colors | ./count_groupby.py
```

### Part 1.b: Analysis

The [PythonWiki has a page on the runtime of python data structures](https://wiki.python.org/moin/TimeComplexity).
The `set.add` method and the `Counter`

The python `set` and `Counter` types are both implemented as 

## Part 2: Pure POSIX, No Python

`uniq` and `sort` commands are included in every POSIX compliant shell.
They are commonly used together to perform "count distinct" and "group by" style queries,
and have different runtime characteristics than the python implementations above.

The `uniq` command filters its input to remove any line that is the same as its previous line.
To see an example, compare the output of the following two commands.
```
$ cat colors
$ cat colors | uniq
```
Notice that every line in the output of `uniq` is not necessarily "unique".

One common strategy for outputting only the distinct lines is to first sort the input.
```
$ cat colors | sort | uniq
```
Now notice that there are no duplicate colors in the output.
We can complete our count distinct query by counting the total number of lines.
```
$ cat colors | sort | uniq | wc -l
```
And we can complete our count groupby query by adding the `-c` flag to the `uniq` command.
```
$ cat colors | sort | uniq -c
```

### Bigger Data

Recall that the file `/data/Twitter dataset/geoTwitter20-01-01.zip` contains all of the geolocated tweets sent on January 1st 2020.
Let's do an analysis to see how many distinct countries sent tweets on this day,
and how many tweets were sent from each of these countries.

First, filter the original json dataset into a list of just country codes.
```
$ time unzip -p /data/Twitter\ dataset/geoTwitter20-01-01.zip | jq '.place.country_code' > country_code
```
This will take about 5 minutes or so.

Then give yourself a high-level overview of your newly formed dataset.
```
$ du -hd0 country_code
$ wc -l country_code
$ head country_code
```
This dataset has the same number of data points (i.e. lines) as our original dataset,
but because data point contains much less information,
it is much smaller and easier to work with.

We can now easily count the total number of distinct countries using either our python/shell combo or in pure shell.
```
$ cat country_code | ./count_distinct.py
$ cat country_code | sort | uniq | wc -l
```

But how would we do the count groupby operation?
The output of both of our code patterns above are hard to interpret.
```
$ cat country_code | ./count_groupby.py
$ cat country_code | sort | uniq -c
```

Modify the `count_groupby.py` file so that it prints the output in sorted order.
You can accomplish this by deleting the final line and replacing it with these three lines.
```
sorted_lines = sorted([(y, x) for (x, y) in unique_lines.items()])
for x, y in sorted_lines:
    print(x, y)
```
Run the command
```
$ cat country_code | sort | uniq -c | sort
```
Notice that the final output does not appear to be sorted.
This is because by default, sort considers all lines as strings and sorts [ASCIIbeticallly](https://en.wiktionary.org/wiki/ASCIIbetical).
In this ordering, the string `      9` comes after the string `   9968` instead of before it.
In this case, that's not desired behavior, and we instead want to sort numerically.
Passing the `-n` flag to `sort` accomplishes this.
```
$ cat country_code | sort | uniq -c | sort -n
```

<!--
## fancy sorting

The `sort` command uses an [external sorting procedure](https://en.wikipedia.org/wiki/External_sorting).
This is essentially merge sort, but 
-->

```
$ cat country_code | sort | uniq -c | sort -n | tail > country_code.plot_data
```

## Plotting with gnuplot

Gnuplot is a popular program for generating charts on the terminal.
It is [the recommended tool for generating plots for wikipedia](https://en.wikipedia.org/wiki/Wikipedia:How_to_create_charts_for_Wikipedia_articles#Plotting),
and [most plots on wikipedia were generated with gnuplot](https://commons.wikimedia.org/wiki/Category:Gnuplot_diagrams).

> **Note:**
>
> The [GNU project](https://www.gnu.org/) is an open source rewrite of the Unix operating system,
> and all of the terminal commands we have used so far are part of GNU.
> [Gnuplot, however, is not affiliated with the GNU project](http://www.gnuplot.info/faq/#x1-120001.7).
> Therefore, even though [GNU is pronounced with a hard G](https://www.gnu.org/gnu/pronunciation.html),
> gnuplot is often pronounced as "newplot" using the standard English pronunciation of gnu.

Start the gnuplot program.
```
$ gnuplot
```
You should get a large welcome message printed followed by a new prompt `gnuplot>` indicating that you are now typing gnuplot commands instead of shell commands.

The simplest command is the `plot` command,
which just takes as input a filename of data to plot.
```
gnuplot> plot 'country_code.plot_data'
```
If you're laptop is a linux machine, then a plot should be displayed on your laptop.
If you're running a Mac or Windows machine, however, running that command will give you an error similar to
```
qt.qpa.screen: QXcbConnection: Could not connect to display 
Could not connect to any X display.
```
(If you got this error on a linux system, you need to reconnect to the lambda server [adding the `-XY` flags](https://explainshell.com/explain?cmd=ssh+-X+-Y+user%40host) to your ssh command.)

The [X Window System](https://en.wikipedia.org/wiki/X_Window_System) is the system used on most Linux machines to display windows to the screen.
It is more advanced than the graphical systems used by Microsoft Windows and Mac because it is designed to allow Windows created by remote machines (for example via an ssh connection).

> **Historical Note:**
>
> Many people think that X Windows gets its name from an allusion to Microsoft Windows graphical interface.
> But the opposite is closer to the truth.
> The first version of X Windows was released in 1984,
> and the first version of Microsoft Windows was released on 1985.

In order for the gnuplot command to display a window on a Mac (or Linux) machine, you will need to download an X server.
[Xming](http://www.straightrunning.com/XmingNotes/) is the most popular one for Windows,
and [XQuartz](https://www.xquartz.org/) for Mac.
For this lab, however, you don't need to install this software if you don't want to.
We'll see alternative ways to get access to the plots.

The first alternative is to plot directly in the terminal window.
This is a good time to mention that the word *terminal* has a different meaning in the context of gnuplot.
Recall that in most contexts, a terminal is the graphical program on your computer that use when interacting with the shell.
In gnuplot, however, a *terminal* refers to the graphical engine used to render the plot.
The default terminal renders the plot to the X Windows system.
We can change the terminal to the [dumb terminal](http://www.gnuplot.info/docs_4.2/node367.html) to get the contents of our plot printed as ASCII art.
```
gnuplot> set terminal dumb
gnuplot> plot 'country_code.plot_data'
  1.1e+06 +----------------------------------------------------------------+
          |      +      +       +      +      +      +       +      +      |
    1e+06 |-+                             'country_code.plot_data'    A  +-|
          |                                                                |
   900000 |-+                                                            +-|
   800000 |-+                                                            +-|
          |                                                                |
   700000 |-+                                                            +-|
          |                                                                |
   600000 |-+                                                            +-|
          |                                                         A      |
   500000 |-+                                                            +-|
          |                                                                |
   400000 |-+                                                            +-|
          |                                                                |
   300000 |-+                                                A           +-|
   200000 |-+                                        A                   +-|
          |                            A      A                            |
   100000 |-+    A      A       A                                        +-|
          |      +      +       +      +      +      +       +      +      |
        0 +----------------------------------------------------------------+
          0      1      2       3      4      5      6       7      8      9
```
We can make this a little bit nicer by adding some more formatting commands:
```
gnuplot> set style data histogram
gnuplot> set style fill solid border -1
gnuplot> plot 'country_code.plot_data' using 1:xtic(2) notitle
  1.1e+06 +----------------------------------------------------------------+
          |     +     +     +     +     +    +     +     +     +     ***   |
    1e+06 |-+                                                        * * +-|
          |                                                          * *   |
   900000 |-+                                                        * * +-|
   800000 |-+                                                        * * +-|
          |                                                          * *   |
   700000 |-+                                                        * * +-|
          |                                                          * *   |
   600000 |-+                                                        * * +-|
          |                                                    ***   * *   |
   500000 |-+                                                  * *   * * +-|
          |                                                    * *   * *   |
   400000 |-+                                                  * *   * * +-|
          |                                                    * *   * *   |
   300000 |-+                                            ***   * *   * * +-|
   200000 |-+                                      ***   * *   * *   * * +-|
          |                             ***  ***   * *   * *   * *   * *   |
   100000 |-+   ***   ***   ***   ***   * *  * *   * *   * *   * *   * * +-|
          |     * *   * *   * *   * *   * *  * *   * *   * *   * *   * *   |
        0 +----------------------------------------------------------------+
               SA    TR    AR    IN    ID   PH    GB    JP    BR    US
```
The `set style` commands change the formatting to use a bar plot instead of a line plot.
The `using 1:xtic(2)` tells gnuplot that the first column in the datafile should be the height of the bars,
and the second column should be the label on the x-axis.
Finally, the `notitle` command removes the legend.

This second plot is a little bit nicer,
but it's still not very good since the results are limited to ASCII art.
To generate a proper plot, we can use the `png` terminal.

The following commands will replot the graph above and store it in the file `top10.png`.
(There is no need to recall the `set style` commands, as those will remain in effect.)
```
gnuplot> set terminal png size 800,400
gnuplot> set output 'top10.png'
gnuplot> plot 'country_code.plot_data' using 1:xtic(2) notitle
```
The plot command above should have no output.
Leave the gnuplot shell by typing `^D` and run `ls`.
You should see the file `top10.png` was created in your lab folder.

> **Note:**
> 
> `png` is [officially pronounced like "ping"](https://en.wikipedia.org/wiki/PNG),
> and so `top10.png` is pronounced as "top ten dot ping".

### Part : Viewing the Image

Unfortunately, there's no way to view graphics inside the terminal.
To view the `top10.png` file, you will need to transfer it to you computer.

On linux machines, this is trivial.
The `sshfs` program lets you [mount](https://unix.stackexchange.com/questions/3192/what-is-meant-by-mounting-a-device-in-linux) the lambda server's filesystem onto your own filesystem.
For example, if I run the following command on my laptop:
```
$ sshfs -p 5055 csci143example@lambda.compute.cmc.edu:/home/csci143example ~/lambda
```
Then the folder `~/lambda` on my laptop will contain all of the contents of my home folder `/home/csci143example` on the lambda server.
I can then navigate to that folder using my standard file explorer tools to view the file.

> **Note:**
>
> There exist sshfs implementations for [Mac](https://osxfuse.github.io) and [Windows](https://github.com/winfsp/sshfs-win).
> You are not required to download and install them,
> but they may make working on the lambda server easier for you.

If you don't have `sshfs` installed, then the next best option is to use github to transfer the file.
Running the commands
```
$ git add top10.png
$ git commit -m 'added top10.png'
$ git push origin master
```
will add the `top10.png` file to your forked repo on github,
and you can view the file there.
If you do that successfully,
then the folling image should display without an error.

<img src=top10.png />

Don't move on to the next steps until you're successfully able to view your image (either through sshfs or github).

### Part : Writing a Gnuplot Script

Working with gnuplot through the terminal interface is possible,
but it's much more convenient to write scripts that generate plots without manual intervention.
We will now see how to write and use these scripts.

Create a file `boxplot.gp` with the following contents.
```
set terminal png size 800,400
set output 'top10.png'
set style data histogram
set style fill solid border -1
plot 'country_code.plot_data' using 1:xtic(2) notitle
```
Notice that these are just the commands that we previously typed directly into the gnuplot terminal.
We can now run all of these commands at once by passing the `-c boxplot.gp` arguments to gnuplot.

First, delete the `top10.png` file.
```
$ rm top10.png
$ ls
```
Then run the script and verify that it worked by seeing that it recreated the file.
```
$ gnuplot -c boxplot.gp
$ ls
```

Just like python functions are more useful when they take parameters that adjust how they work,
scripts are also more useful when they take input.
We will modify the script so that it can be used with the pipe.
This will require two changes:
1. Replace the hard coded output filename `'top10.png'` with the variable `ARG1`.
    Gnuplot will substitute the first command line argument of the script with this value.
1. Replace the hard coded input filename `'country_code.plot_data'` with the special filename `'/dev/stdin'`.
    This will allow us to get our data from the pipe.

After making these changes, the final `boxplot.gp` script should look like:
```
set terminal png size 800,400
set output ARG1
set style data histogram
set style fill solid border -1
plot '/dev/stdin' using 1:xtic(2) notitle
```
To test this new script, we will recreate the original `top10.png` file.
```
$ rm top10.png
$ ls
$ cat 'country_code.plot_data' | gnuplot -c boxplot.gp top10.png
$ ls
```
As before, you should verify that the output of the first `ls` and second `ls` differ only by the newly created `top10.png` file.
```
$ unzip -p /data/Twitter\ dataset/geoTwitter20-01-01.zip | jq '.place.country_code' | sort | uniq -c | sort -n | tail -n10 | gnuplot -c boxplot.gp top10.png
```
```
$ unzip -p /data/Twitter\ dataset/geoTwitter20-01-01.zip \
| jq '.place.country_code' \
| sort \
| uniq -c \
| sort -n \
| tail -n10 \
| gnuplot -c boxplot.gp top10.png
```

## Submission


**Problem 1:**
Plot the number of 

```
$ 
```

<img src=plot1.png>

**Problem 2:**

```
$ 
```

<img src=plot1.png>
