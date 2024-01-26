# Lab: Gnuplot

Recall that in last week's lab, we saw how to use python and SQL to perform the *count distinct* and *count group* queries.
We had two main takeaways:

1. Python's pandas library requires $\Omega(n)$ memory,
    and so it is not suitable for large datasets.
1. SQLite requires only $O(1)$ memory,
     and so is suitable for large datasets.

In this lab, we will not see how to run those same queries in the shell.
We will then see how to make bar charts out of those queries using the terminal program gnuplot.

## Part 0: Setup

Fork this repo, clone your fork onto the lambda server, and enter the repo directory.
It will be important later that you are working on your forked repo because you'll be uploading images to the repo,
and you don't have permission to do that to my repo.

<!--
## Part 1: Combining Python with the shell

The file `colors` in this repo contains a small dataset of different colors.
Examine it.
```
$ cat colors
```
In this section, we will write some short python code to perform the count distinct and count group queries.

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

A similar python program can be used to perform the count group operation.
All we have to do is replace the `set` with `Counter`.

Create a file `count_group.py` with the following code,
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
$ cat colors | ./count_group.py
```

### Part 1.b: Analysis

The [PythonWiki has a page on the runtime of python data structures](https://wiki.python.org/moin/TimeComplexity).
The `set.add` method and the `Counter`

The python `set` and `Counter` types are both implemented as 
-->

## Part 1: Count Distinct/Group in the Shell

The `uniq` and `sort` commands are included in every POSIX compliant operating system.
They are commonly used together to perform the count distinct and count group operations.

### Part 1.a: Count Distinct

The `uniq` command (pronounced like "unique") filters its input to remove any line that is the same as its previous line.
We'll use the `colors` file included in the repo as an example.
Compare the output of the following two commands.
```
$ cat colors
$ cat colors | uniq
```
Notice that every line in the output of `uniq` is not necessarily "unique",
but no two neighboring lines are the same.

One common strategy for outputting only the distinct lines is to first sort the input.
```
$ cat colors | sort | uniq
```
Now notice that there are no duplicate colors in the output.
We can complete our count distinct query by combining with `wc -l`:
```
$ cat colors | sort | uniq | wc -l
```

### Part 1.b: Count Group

The count group query is almost as easy to do.
The `uniq` program with the `-c` flag counts the total number of duplicated rows,
and so adding this flag performs the count group query.
```
$ cat colors | sort | uniq -c
```
Notice, however, that the output above is not sorted by the number of occurrences.
But sorted output would be easier to read, so let's figure out how to get it.

You might think that another call to `sort` would do the trick.
Try it:
```
$ cat colors | sort | uniq -c | sort
```
Technically, this output is now sorted [ASCIIbeticallly](https://en.wiktionary.org/wiki/ASCIIbetical).
(The `1` character comes before ` ` in ASCII, and so `11` comes before ` 1`.)
But this isn't what we really wanted.
We want to sort on the numerical values of the numbers in our table,
and not their ASCII values.
`sort` has a flag `-n` for this purpose.
Adding it to our command, we can now perform a nice count group query on `colors` with the command
```
$ cat colors | sort | uniq -c | sort -n
```

## Part 2: Bigger Data

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
but because each data point contains much less information,
it is much smaller and easier to work with.

> **Exercise:**
> 
> Write a one line shell command for computing the count distinct query on the `country_code` file above.

> **Exercise:**
> 
> Write a one line shell command for computing the count group query on the `country_code` file above.
> Additionally, use the `tail` command to extract the 10 largest country/value combinations, and store the results in a file `country_code.plot_data`.
<!--
```
$ cat country_code | sort | uniq -c | sort -n | tail > country_code.plot_data
```
-->

We will not see how to plot and visualize this data entirely in the shell.

<!--
We can now easily count the total number of distinct countries using either our python/shell combo or in pure shell.
```
$ cat country_code | ./count_distinct.py
$ cat country_code | sort | uniq | wc -l
```

But how would we do the count group operation?
The output of both of our code patterns above are hard to interpret.
```
$ cat country_code | ./count_group.py
$ cat country_code | sort | uniq -c
```

Modify the `count_group.py` file so that it prints the output in sorted order.
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
-->

<!--
## fancy sorting

The `sort` command uses an [external sorting procedure](https://en.wikipedia.org/wiki/External_sorting).
This is essentially merge sort, but 
-->

## Plotting with Gnuplot

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
> [The authors have a detailed FAQ answer about the origin of the name.](http://www.gnuplot.info/faq/#x1-70001.2):

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
If your laptop is a linux machine, then a plot should be displayed on your screen.
If you're running a Mac or Windows machine, however, running that command will give you an error similar to
```
qt.qpa.screen: QXcbConnection: Could not connect to display 
Could not connect to any X display.
```
(If you got this error on a linux system, you can get rid of it by [adding the `-XY` flags](https://explainshell.com/explain?cmd=ssh+-X+-Y+user%40host) to your ssh command.)

This error message references the [X Window System](https://en.wikipedia.org/wiki/X_Window_System),
which is a popular system for displaying graphics on Unix machines.
One of its main advantages is that it allows windows created by remote machines (like the lambda server) to be displayed on your local machine.

> **Historical Note:**
>
> Many people think that X Windows gets its name from an allusion to Microsoft Windows graphical interface.
> But the opposite is closer to the truth.
> The first version of X Windows was released in 1984,
> and the first version of Microsoft Windows was released on 1985.

In order for the gnuplot `plot` command to display a window on a Mac or Windows machine, you will need to download software called an "X server".
[Xming](http://www.straightrunning.com/XmingNotes/) is the most popular one for Windows,
and [XQuartz](https://www.xquartz.org/) for Mac.
For this lab, however, you don't need to install this software if you don't want to.
We'll see alternative ways to get access to the plots.

The first alternative is to plot directly in the terminal window using ASCII art.
This is a good time to mention that the word *terminal* has a different meaning in the context of gnuplot.
Recall that in most contexts, a terminal is the graphical program on your computer that use when interacting with the shell.
In gnuplot, however, a *terminal* refers to the graphical engine used to render the plot.
The default terminal renders the plot to the X Windows system.
We can change the terminal to the [dumb terminal](http://www.gnuplot.info/docs_4.2/node367.html) to get the contents of our plot printed as ASCII art.

The following commands should work for everyone.
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
We can make this plot a little bit nicer by adding some more formatting commands.
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
(There is no need to retype the `set style` commands, as those will remain in effect.)
```
gnuplot> set terminal png size 800,400
gnuplot> set output 'top10.png'
gnuplot> plot 'country_code.plot_data' using 1:xtic(2) notitle
```
The `plot` command above should have no output.
Leave the gnuplot shell by typing `^D` and run `ls`.
You should see the file `top10.png` was created in your lab folder.

> **Note:**
> 
> `png` is [officially pronounced like "ping"](https://en.wikipedia.org/wiki/PNG),
> and so `top10.png` is pronounced as "top ten dot ping".

### Part : Viewing the Image

Unfortunately, there's no way to view graphics inside the terminal.
To view the `top10.png` file, you will need to transfer it to you computer.

On Linux machines, this is trivial.
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
You can upload the file to github with the following commands.
```
$ git add top10.png
$ git commit -m 'added top10.png'
$ git push origin master
```
If these commands worked successfully,
then the image below should work.

<img src=top10.png />

If not, ensure that you're looking at your forked repo and not my repo.

Don't move on to the next steps until you're successfully able to view your image.

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

Armed with this script, we can write a "simple" shell 1-liner that generates this plot from the original data.
```
$ unzip -p /data/Twitter\ dataset/geoTwitter20-01-01.zip | jq '.place.country_code' | sort | uniq -c | sort -n | tail -n10 | gnuplot -c boxplot.gp top10.png
```
For long commands like this, it is common to break them up onto multiple lines.
In the shell (and most programming environments), any line that ends with a backslash `\` is treated as continuing on to the next line.
Thus the following more readable shell command is 100% equivalent to the shell command above.
```
$ unzip -p /data/Twitter\ dataset/geoTwitter20-01-01.zip \
| jq '.place.country_code' \
| sort \
| uniq -c \
| sort -n \
| tail -n10 \
| gnuplot -c boxplot.gp top10.png
```
We can also combine this command with other standard shell syntax like for loops, the glob `*`, and background processing `&`.
The following command generates 
```
$ for file in /data/Twitter\ dataset/geoTwitter20-*-01.zip; do 
    unzip -p "$file" \
    | jq '.place.country_code' \
    | sort \
    | uniq -c \
    | sort -n \
    | tail -n10 \
    | gnuplot -c boxplot.gp $(basename "$file")-top10.png &
done
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
