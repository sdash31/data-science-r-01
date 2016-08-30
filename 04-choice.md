
---
layout: lesson
title: Introduction to Data Science using R
subtitle: Making Choices
minutes: 30
---

> ## Learning objectives {.objectives}
> * Save plot(s) in a PDF file
> * Write conditional statements with `if` and `else`
> * Correctly evaluate expressions containing `&` ('and') and `|` ('or')

Our previous lessons have shown us how to manipulate data, define our own functions, and repeat things.
However, the programs we have written so far always do the same things, regardless of what data they're given.
We want programs to make choices based on the values they are manipulating.

### Saving Plots to a File

So far, we have built a function `analyze` to plot summary statistics of the inflammation data:


```R
setwd("/home/lngo/intro-data-science/")
analyze <- function(filename) {
  # Plots the average, min, and max inflammation over time.
  # Input is character string of a csv file.
  dat <- read.csv(file = filename, header = FALSE)
  avg_day_inflammation <- apply(dat, 2, mean)
  plot(avg_day_inflammation)
  max_day_inflammation <- apply(dat, 2, max)
  plot(max_day_inflammation)
  min_day_inflammation <- apply(dat, 2, min)
  plot(min_day_inflammation)
}
```

And also built the function `analyze_all` to automate the processing of each data file:


```R
analyze_all <- function(pattern) {
  # Runs the function analyze for each file in the current working directory
  # that contains the given pattern.
  filenames <- list.files(path = "data", pattern = pattern, full.names = TRUE)
  for (f in filenames) {
    analyze(f)
  }
}
```

While these are useful in an interactive R session, what if we want to send our results to our collaborators?
Since we currently have 12 data sets, running `analyze_all` creates 36 plots.
Saving each of these individually would be tedious and error-prone.
And in the likely situation that we want to change how the data is processed or the look of the plots, we would have to once again save all 36 before sharing the updated results with our collaborators.

Here's how we can save all three plots of the first inflammation data set in a pdf file:


```R
pdf("inflammation-01.pdf")
analyze("data/inflammation-01.csv")
dev.off()
```

The function `pdf` redirects all the plots generated by R into a pdf file, which in this case we have named "inflammation-01.pdf".
After we are done generating the plots to be saved in the pdf file, we stop R from redirecting plots with the function `dev.off`.

> ## Overwriting Plots
>
> If you run `pdf` multiple times without running `dev.off`, you will save plots to the most recently opened file.
> However, you won't be able to open the previous pdf files because the connections were not closed.
> In order to get out of this situation, you'll need to run `dev.off` until all the pdf connections are closed.
> You can check your current status using the function `dev.cur`.
> If it says "pdf", all your plots are being saved in the last pdf specified.
> If it says "null device" or "RStudioGD", the plots will be visualized normally.


We can update the `analyze` function so that it always saves the plots in a pdf.
But that would make it more difficult to interactively test out new changes.
It would be ideal if `analyze` would either save or not save the plots based on its input.

### Conditionals

In order to update our function to decide between saving or not, we need to write code that automatically decides between multiple options.
The tool R gives us for doing this is called a conditional statement, and looks like this:


```R
num <- 37
if (num > 100) {
  print("greater")
} else {
  print("not greater")
}
print("done")
```

The second line of this code uses an `if` statement to tell R that we want to make a choice.
If the following test is true, the body of the `if` (i.e., the lines in the curly braces underneath it) are executed.
If the test is false, the body of the `else` is executed instead.
Only one or the other is ever executed.

In the example above, the test `num > 100` returns the value `FALSE`, which is why the code inside the `if` block was skipped and the code inside the `else` statement was run instead.


```R
num > 100
```

And as you likely guessed, the opposite of `FALSE` is `TRUE`.


```R
num < 100
```

Conditional statements don't have to include an `else`.
If there isn't one, R simply does nothing if the test is false:


```R
num <- 53
if (num > 100) {
  print("num is greater than 100")
}
```

We can also chain several tests together when there are more than two options.
This makes it simple to write a function that returns the sign of a number:


```R
sign <- function(num) {
  if (num > 0) {
    return(1)
  } else if (num == 0) {
    return(0)
  } else {
    return(-1)
  }
}

sign(-3)

sign(0)

sign(2/3)
```

Note that when combining `else` and `if` in an `else if` statement (similar to `elif` in Python), the `if` portion still requires a direct input condition.  This is never the case for the `else` statement alone, which is only executed if all other conditions go unsatisfied.
Note that the test for equality uses two equal signs, `==`.

> ## Other Comparisons
>
> Other tests include greater than or equal to (`>=`), less than or equal to
> (`<=`), and not equal to (`!=`).

We can also combine tests.
An ampersand, `&`, symbolizes "and".
A vertical bar, `|`, symbolizes "or".
`&` is only true if both parts are true:


```R
if (1 > 0 & -1 > 0) {
    print("both parts are true")
} else {
  print("at least one part is not true")
}
```

while `|` is true if either part is true:


```R
if (1 > 0 | -1 > 0) {
    print("at least one part is true")
} else {
  print("neither part is true")
}
```

In this case, "either" means "either or both", not "either one or the other but not both".

> ## Choosing Plots Based on Data
>
> Write a function `plot_dist` that plots
> a boxplot if the length of the vector is greater than a specified threshold
> and a stripchart otherwise.
> To do this you'll use the R functions `boxplot` and `stripchart`.


```R
dat <- read.csv("data/inflammation-01.csv", header = FALSE)
plot_dist(dat[, 10], threshold = 10)     # day (column) 10

plot_dist(dat[1:5, 10], threshold = 10)  # samples (rows) 1-5 on day (column) 10
```

> ## Histograms Instead
>
> One of your collaborators prefers to see the distributions of the larger vectors
> as a histogram instead of as a boxplot.
> In order to choose between a histogram and a boxplot
> we will edit the function `plot_dist` and add an additional argument `use_boxplot`.
> By default we will set `use_boxplot` to `TRUE`
> which will create a boxplot when the vector is longer than `threshold`.
> When `use_boxplot` is set to `FALSE`,
> `plot_dist` will instead plot a histogram for the larger vectors.
> As before, if the length of the vector is shorter than `threshold`,
> `plot_dist` will create a stripchart.
> A histogram is made with the `hist` command in R.


> ## Find the Maximum Inflammation Score
>
> Find the file containing the patient with the highest average inflammation score.
> Print the file name, the patient number (row number) and the value of the maximum average inflammation score.
>
> Tips:
>
> 1. Use variables to store the maximum average and update it as you go through files and patients.
> 1. You can use nested loops (one loop is inside the other) to go through the files as well as through the patients in each file (every row).
>
> Complete the code below:
>
> 
> filenames <- list.files(path = "data", pattern = "inflammation.*csv", full.names = TRUE)
> filename_max <- "" # filename where the maximum average inflammation patient is found
> patient_max <- 0 # index (row number) for this patient in this file
> average_inf_max <- 0 # value of the average inflammation score for this patient
> for (f in filenames) {
>   dat <- read.csv(file = f, header = FALSE)
>   dat.means = apply(dat, 1, mean)
>   for (patient_index in length(dat.means)){
>     patient_average_inf = dat.means[patient_index]
>     # Add your code here ...
>   }
> }
> print(filename_max)
> print(patient_max)
> print(average_inf_max)

### Saving Automatically Generated Figures

Now that we know how to have R make decisions based on input values,
let's update `analyze`:


```R
analyze <- function(filename, output = NULL) {
  # Plots the average, min, and max inflammation over time.
  # Input:
  #    filename: character string of a csv file
  #    output: character string of pdf file for saving
  if (!is.null(output)) {
    pdf(output)
  }
  dat <- read.csv(file = filename, header = FALSE)
  avg_day_inflammation <- apply(dat, 2, mean)
  plot(avg_day_inflammation)
  max_day_inflammation <- apply(dat, 2, max)
  plot(max_day_inflammation)
  min_day_inflammation <- apply(dat, 2, min)
  plot(min_day_inflammation)
  if (!is.null(output)) {
    dev.off()
  }
}
```

We added an argument, `output`, that by default is set to `NULL`.
An `if` statement at the beginning checks the argument `output` to decide whether or not to save the plots to a pdf.
Let's break it down.
The function `is.null` returns `TRUE` if a variable is `NULL` and `FALSE` otherwise.
The exclamation mark, `!`, stands for "not".
Therefore the line in the `if` block is only executed if `output` is "not null".


```R
output <- NULL
is.null(output)
!is.null(output)
```

Now we can use `analyze` interactively, as before,


```R
analyze("data/inflammation-01.csv")
```

but also use it to save plots,


```R
analyze("data/inflammation-01.csv", output = "inflammation-01.pdf")
```

Before going further, we will create a directory `results` for saving our plots.
It is good practice in data analysis projects to save all output to a directory separate from the data and analysis code.
You can create this directory using the shell command `mkdir`, or the R function `dir.create()`


```R
dir.create("results")
```

Now run `analyze` and save the plot in the `results` directory,


```R
analyze("data/inflammation-01.csv", output = "results/inflammation-01.pdf")
```

This now works well when we want to process one data file at a time, but how can we specify the output file in `analyze_all`?
We need to do two things:

1. Substitute the filename ending "csv" with "pdf".
2. Save the plot to the `results` directory.

To change the extension to "pdf", we will use the function `sub`,


```R
f <- "inflammation-01.csv"
sub("csv", "pdf", f)
```




'inflammation-01.pdf'



To add the "data" directory to the filename use the function `file.path`,


```R
file.path("results", sub("csv", "pdf", f))
```




'results/inflammation-01.pdf'



Now let's update `analyze_all`:


```R
analyze_all <- function(pattern) {
  # Directory name containing the data
  data_dir <- "data"
  # Directory name for results
  results_dir <- "results"
  # Runs the function analyze for each file in the current working directory
  # that contains the given pattern.
  filenames <- list.files(path = data_dir, pattern = pattern)
  for (f in filenames) {
    pdf_name <- file.path(results_dir, sub("csv", "pdf", f))
    analyze(file.path(data_dir, f), output = pdf_name)
  }
}

analyze_all("inflammation*.csv")
```

Now if we need to make any changes to our analysis, we can edit the `analyze` function and quickly regenerate all the figures with `analyze_all`.

> ## Changing the Behavior of the Plot Command
>
> One of your collaborators asks if you can recreate the figures with lines instead of points.
> Find the relevant argument to `plot` by reading the documentation (`?plot`),
> update `analyze`, and then recreate all the figures with `analyze_all`.