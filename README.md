# Lab: POSIX MapReduce

<!--
REMINDER TO MYSELF:
The `notes` branch contains some partially completed lab material.
-->

Recall that in last week's lab, we saw how to use python and SQL to perform the *count distinct* and *count group* queries.
We had two main takeaways:

1. Python's pandas library requires $\Omega(n)$ memory,
    and so it is not suitable for large datasets.
1. SQLite requires only $O(1)$ memory,
     and so is suitable for large datasets.

In this lab, we will see how to run those same queries in the shell.
We'll also see how to make nice visualizations of those queries using the terminal program gnuplot,
and parallelize those queries using MapReduce.

## Part 0: Setup

Fork this repo, clone your fork onto the lambda server, and cd into the repo directory.
It will be important later that you are working on your forked repo because you'll be uploading images to the repo for your submission.

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

> **Note:**
> Unfortunately, the `sort` command above is rather expensive.
> Sorting requires $\Omega(n)$ memory and $\Omega(n\log n)$ compute,
> and the `sort` command is the only commonly used shell command that does not use `O(1)` memory.
> Still, the `sort` command is about as efficient as theoretically possible.
> It uses an [external sort algorithm](https://en.wikipedia.org/wiki/External_sorting),
> which is something that you probably did not study in data structures.
> External sort is very similar to merge sort, except that the intermediate steps are stored on the hard drive.
> This allows `sort` to sort extremely large files even on systems with limited RAM.

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

> **Exercise:**
> Use output redirection to store the output of the command above in a file called `colors.dat`.
> We will use this file to experiment with plotting in the next section.

## Part 2: Plotting with Gnuplot

It is common to plot the results of a count group query as a bar chart.
In this section, we'll see how to do that with gnuplot.
Gnuplot is a popular program for generating charts on the terminal.
It is [the recommended tool for generating plots for wikipedia](https://en.wikipedia.org/wiki/Wikipedia:How_to_create_charts_for_Wikipedia_articles#Plotting),
and [most plots on wikipedia were generated with gnuplot](https://commons.wikimedia.org/wiki/Category:Gnuplot_diagrams).

> **Note:**
> The [GNU project](https://www.gnu.org/) is an open source rewrite of the Unix operating system,
> and all of the terminal commands we have used so far are part of GNU.
> [Gnuplot, however, is not affiliated with the GNU project](http://www.gnuplot.info/faq/#x1-120001.7).
> Therefore, even though [GNU is pronounced with a hard G](https://www.gnu.org/gnu/pronunciation.html),
> gnuplot is canonically pronounced as "newplot" using the standard English pronunciation of gnu.
> [The authors have a detailed FAQ answer about the origin of the name.](http://www.gnuplot.info/faq/#x1-70001.2):

### Part 2.a: A First Attempt at Plotting

Start the gnuplot program.
```
$ gnuplot
```
You should get a large welcome message printed followed by a new prompt `gnuplot>` indicating that you are now typing gnuplot commands instead of shell commands.

The simplest command is the `plot` command,
which just takes as input a filename of data to plot.
Try it.
```
gnuplot> plot 'colors.dat'
```
You likely get an error similar to
```
qt.qpa.screen: QXcbConnection: Could not connect to display 
Could not connect to any X display.
```
This error message references the [X Window System](https://en.wikipedia.org/wiki/X_Window_System),
which is a popular system for displaying graphics on Linux machines.
One of its main advantages is that it allows windows created by remote machines (like the lambda server) to be displayed on your local machine.

If your laptop is a linux machine, then you already have X Windows installed.
You can solve this problem by enabling "X forwarding" with the [`-XY` flags](https://explainshell.com/explain?cmd=ssh+-X+-Y+user%40host) in your ssh command.

There exist open source X Windows implementations for every operations system.
[Xming](http://www.straightrunning.com/XmingNotes/) is the most popular one for Windows,
and [XQuartz](https://www.xquartz.org/) for Mac.
If you installed one of these on your laptop, then you would also be able to open windows on the lambda server and have them appear on your machine.
For this lab, however, you don't need to install this software if you don't want to.
We'll see alternative ways to get access to the plots.

> **Note:**
> Many people think that X Windows gets its name from an allusion to Microsoft Windows graphical interface.
> But the opposite is closer to the truth.
> The first version of X Windows was released in 1984,
> and the first version of Microsoft Windows was released on 1985.

### Part 2.b: Plotting with ASCII Art

The easiest way to view the plots is to plot them directly in the terminal with ASCII art.

This is a good time to mention that the word *terminal* has a different meaning in the context of gnuplot.
Recall that in most contexts, a terminal is the graphical program on your computer that use when interacting with the shell.
In gnuplot, however, a [*terminal* refers to the graphical engine used to render the plot](http://gnuplot.info/docs_5.5/Terminals.html).
The [default terminal is called `qt`](http://gnuplot.info/docs_5.5/loc21993.html) because it uses the [QT library](https://en.wikipedia.org/wiki/Qt_(software)) to render the plot to the X Windows system.
We can change the terminal to the [dumb terminal](http://www.gnuplot.info/docs_4.2/node367.html) to get the contents of our plot printed as ASCII art.

The following commands should work for everyone.
```
gnuplot> set terminal dumb
gnuplot> plot 'colors.dat'
  12 +---------------------------------------------------------------------+
     |                      +                       +                      |
     |                                                                     |
  10 |-+                                                                 +-|
     |                                                                     |
     |                                                                     |
     |                                                                     |
   8 |-+                                                                 +-|
     |                                                                     |
     |                                                                     |
   6 |-+                                                                 +-|
     |                                                                     |
     |                                                                     |
   4 |-+                                            A                    +-|
     |                                                                     |
     |                      A                                              |
     |                                                                     |
   2 |-+                                                                 +-|
     |                                                                     |
     |                      +                       +                      |
   0 +---------------------------------------------------------------------+
  yellow                  green                    red                   blue
```
This plot is hard to read, and the values for `yellow` and `blue` aren't even being displayed because they overlap with the axes.
We can make this plot a little bit nicer by adding some more formatting commands.
```
gnuplot> set style data histogram
gnuplot> set style fill solid border -1
gnuplot> plot 'colors.dat' using 1:xtic(2) notitle
  12 +---------------------------------------------------------------------+
     |             +             +             +             +             |
     |                                                       ******        |
  10 |-+                                                     *    *      +-|
     |                                                       *    *        |
     |                                                       *    *        |
     |                                                       *    *        |
   8 |-+                                                     *    *      +-|
     |                                                       *    *        |
     |                                                       *    *        |
   6 |-+                                                     *    *      +-|
     |                                                       *    *        |
     |                                                       *    *        |
   4 |-+                                       ******        *    *      +-|
     |                                         *    *        *    *        |
     |                           ******        *    *        *    *        |
     |                           *    *        *    *        *    *        |
   2 |-+                         *    *        *    *        *    *      +-|
     |             ******        *    *        *    *        *    *        |
     |             *    *        *    *        *    *        *    *        |
   0 +---------------------------------------------------------------------+
                yellow         green          red          blue
```
The `set style` commands change the formatting to use a bar plot instead of a line plot.
The `using 1:xtic(2)` tells gnuplot that the first column in the datafile should be the height of the bars,
and the second column should be the label on the x-axis.
Finally, the `notitle` command removes the legend.

### Part 2.c: Generating PNG Files

This second plot is a little bit nicer,
but it's still not very good since the results are limited to ASCII art.
To generate a proper plot, we can use the `png` terminal.

The following commands will replot the graph above and store it in the file `colors.png`.
(There is no need to retype the `set style` commands, as those will remain in effect.)
```
gnuplot> set terminal png size 800,400
gnuplot> set output 'colors.png'
gnuplot> plot 'country_code.plot_data' using 1:xtic(2) notitle
```
The `plot` command above should have no output.
Leave the gnuplot shell by typing `^D` and run `ls`.
You should see the file `colors.png` was created in your lab folder.

> **Note:**
> `png` is [officially pronounced like "ping"](https://en.wikipedia.org/wiki/PNG),
> and so `colors.png` is pronounced as "top ten dot ping".

### Part 2.d: Viewing the Image

Unfortunately, there's no way to view png files inside the terminal.
To view the `colors.png` file, you will need to transfer it to you computer.

On Linux machines, this is once again trivial.
The `sshfs` program lets you [mount](https://unix.stackexchange.com/questions/3192/what-is-meant-by-mounting-a-device-in-linux) the lambda server's filesystem onto your own filesystem.
For example, if I run the following command on my laptop:
```
$ sshfs -p 5055 csci143example@lambda.compute.cmc.edu:/home/csci143example ~/lambda
```
Then the folder `~/lambda` on my laptop will contain all of the contents of my home folder `/home/csci143example` on the lambda server.
I can then navigate to that folder using my standard file explorer tools to view the file.

> **Note:**
> There exist sshfs implementations for [Mac](https://osxfuse.github.io) and [Windows](https://github.com/winfsp/sshfs-win).
> You are not required to download and install them,
> but they may make working on the lambda server easier for you.

If you don't have `sshfs` installed, then the next best option is to use github to transfer the file.
You can upload the file to github with the following commands.
```
$ git add colors.png
$ git commit -m 'added colors.png'
$ git push origin master
```
If these commands worked successfully,
then the image below should work.

<img src=colors.png />

If not, ensure that you're looking at your forked repo and not my repo.

Don't move on to the next steps until you're successfully able to view your image.
Part of the submission for this lab will require that all of these broken image links are replaced with working images.

## Part 3: Writing Gnuplot Scripts

Working with gnuplot through the terminal interface is possible,
but it's annoying to have to retype all of those commands anytime we want to make a plot.
A better solution is to automate plotting with a script.
We will now see how to write and use these scripts.

Create a file `boxplot.gp` with the following contents.
```
set terminal png size 800,400
set output 'colors.png'
set style data histogram
set style fill solid border -1
plot 'country_code.plot_data' using 1:xtic(2) notitle
```
Notice that these are just the commands that we previously typed directly into the gnuplot terminal.
We can now run all of these commands at once by passing the `-c boxplot.gp` arguments to gnuplot.

First, delete the `colors.png` file.
```
$ rm colors.png
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

1. Replace the hard coded output filename `'colors.png'` with the variable `ARG1`.
    Gnuplot will substitute the first command line argument of the script with this value.
1. Replace the hard coded input filename `'colors.dat'` with the special filename `'/dev/stdin'`.
    This will allow us to get our data from the pipe.

After making these changes, the final `boxplot.gp` script should look like:
```
set terminal png size 800,400
set output ARG1
set style data histogram
set style fill solid border -1
plot '/dev/stdin' using 1:xtic(2) notitle
```
To test this new script, we will recreate the original `colors.png` file.
```
$ rm colors.png
$ ls
$ cat 'colors.dat' | gnuplot -c boxplot.gp colors.png
$ ls
```
As before, you should verify that the output of the first `ls` and second `ls` differ only by the newly created `colors.png` file.

## Part 4: Bigger Data

Recall that the file `/data/Twitter dataset/geoTwitter20-01-01.zip` contains all of the geolocated tweets sent on January 1st 2020.
Let's do an analysis to see how many tweets were sent from each country on this day.
We can easily do this with the following shell 1-liner:
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
This command takes about 5 minutes to run.
Perhaps surprisingly, the bottleneck of this command is the `jq` command which parses the JSON.
To verify this, press `^Z` while the program above is running,
then run the command `ps`.
You should get output similar to
```
$ ps
  PID TTY          TIME CMD
10327 pts/3    00:00:00 bash
25755 pts/3    00:00:22 unzip
25756 pts/3    00:00:54 jq
25757 pts/3    00:00:00 sort
25758 pts/3    00:00:00 uniq
25759 pts/3    00:00:00 sort
25760 pts/3    00:00:00 tail
25761 pts/3    00:00:00 gnuplot
25825 pts/3    00:00:00 ps
```
Recall that `ps` lists all of the processes that are currently running.
Notice that each of the commands in your shell 1-liner is actually running concurrently.
The time that is listed for each of these processes is the total amount of CPU time that process has used.
In the output above, `unzip` has used 22 seconds, and `jq` has used 54 seconds.
Fortunately, because the lambda server has so many CPUs available, each of these processes will be running on their own CPU and running in parallel.
The amount of time for the entire command to complete is therefore only the length of time for the slowest command,
and not the total length of time for all commands.
In Python, it is essentially impossible to have the unzip and json decoding happen in parallel due to the [Global Interpreter Lock (GIL)](https://wiki.python.org/moin/GlobalInterpreterLock) (although there is [some recent work to fix this peoblem](https://www.infoworld.com/article/3704248/python-moves-to-remove-the-gil-and-boost-concurrency.html)).
In the shell, it is trivial to have these expensive tasks run in parallel.

When your command completes, upload the `top10.png` file to github.
You should see it appear below.

<img src=top10.png />

## Part 5: A Simple MapReduce

Recall from the [MapReduce homework](https://github.com/mikeizbicki/twitter_coronavirus) that MapReduce is a parallel procedure for large scale data analysis.
In MapReduce, the "mappers" analyze small parts of the dataset in parallel, and then the "reducers" combine those results into a final result.
In the homework, you used (or will use) a combination of python and the shell to perform these tasks.
In this lab, we'll see how to perform MapReduce entirely in the shell.

Our goal will be to generate a plot of how many tweets were sent from each country in the first 9 days of 2020.

The mapper is fairly simple.
It's just a count group query like we did in the previous section, but without plotting the results.
The following shell code runs these mappers in parallel.
```
$ for file in /data/Twitter\ dataset/geoTwitter20-01-0*.zip; do 
unzip -p "$file" \
| jq '.place.country_code' \
| sort \
| uniq -c \
| sort -n \
> map.$(basename "$file").dat &
done
```

The reducer is more complicated.
The code below performs the reduce step by merging all of the  outputs from the map step into a single file.
```
cat map.geoTwitter20-01-01.zip.dat | while read line; do
    country_code=$(echo "$line" | sed 's/[^a-zA-Z"]//g')
    counts=$(cat map.* | grep "$country_code" | sed 's/[^0-9]//g')
    sum=$(echo $counts | sed 's/ /+/g' | bc)
    echo "$sum" "$country_code"
done | sort -n > reduce
```
The while loop above reads each line from stdin (i.e. the output of the `cat` command) one at a time, storing it in the `line` variable.
We then extract the country code, search all of the map files for that country code, and sum their totals together with the `bc` command (bc stands for basic calculator and is the main tool for doing math in the shell).
The final output is re-sorted and stored in the file `reduce`.

> **Exercise:**
> Run the map and reduce procedures above.
> Then create a plot of the results in the file `country_code_mapreduce.png`.
> Upload your plot to github and ensure that it appears below.

<img src=country_code_mapreduce.png />

## Part 6: MapReduce Challenge

In this last section, you will have to write your own MapReduce procedure based on the code above.
I want you to calculate how many tweets sent from the United States are written in each language over the period Jan 1 to Jan 9 2020.

> **Hint:**
> You should modify the above commands to filter out tweets that don't come from the US with the `grep` command.
> You should also modify the command to do a count group by query on the language the tweet was written in.
> You'll have to examine the JSON objects in order to figure out where this information is stored.

Plot the top 20 languages into the file `uslang-top20.png` and upload it to github.

<img src=uslang-top20.png />

## Submission

Upload the url of your forked github repo to sakai.
In order to get full credit for the lab,
you'll need to have all of the images uploaded to github.

The lab is worth 4 points.
Part 6 is worth 2 points, and the rest of the lab the other 2 points.
