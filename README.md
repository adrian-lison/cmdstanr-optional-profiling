# Optional profiling in CmdStanR <img src="logo.png" align="right" width="120" />

This repository provides a wrapper for the `cmdstanr::cmdstan_model` method to make profiling of a Stan program *optional*.

## Why make profiling optional?
Profiling is a great tool for debugging and performance optimization. It helps to identify which parts of a Stan program are taking the longest time to run. But it also adds a bit of overhead to the execution of the Stan program, so it would be nice to skip the profiling statements whenever you don't need them and want the model to run as fast as possible.

Currently, there is no native support for deactivating the profiling on a Stan program with profile statements. The only workaround is to make a copy of your Stan model file without the profile statements and run it instead. This can obviously become very tedious, especially if you distribute your model over different files using include statements. For exactly this reason, I have written the wrapper in this repository to do all the work for you.

## How can I use it?
I am too lazy to make this a package (and hope either this or a different solution will be implemented in CmdStanR soon), so currently you have to manually download [the R script](cmdstan_model_optional_profiling.R) in this repository and source it in R:

```
source("cmdstan_model_optional_profiling.R")
```

Then, if you have a model you want to run using CmdStanR:

```
example_file <- file.path(cmdstan_path(), "examples/bernoulli/bernoulli.stan")
```

and want to compile the model without profiling, you can either use this function:

```
cmdstan_model_no_profiling(example_file)
```

or this function:

```
cmdstan_model_optional_profiling(example_file, profile = FALSE)
```

Both functions behave exactly as the original `cmdstanr::cmdstan_model`. You can supply the same additional arguments, in particular also `include_paths`, if you include Stan files from a different directory than the one of the main model.

## How does it work?
Under the hood, this is really just a workaround. All relevant Stan model files
are copied to `tempdir()` and all profile statements are removed using regex. The regex should work reliably on a valid Stan model, but could fail or produce weird compilation errors on invalid Stan models. So if you have problems, try to compile with the normal `cmdstanr::cmdstan_model` or with  `cmdstan_model_optional_profiling(..., profile=TRUE)` first. If that is working, but the regex still fails, I'd be interested to know about it.

One thing to note is that not only the required Stan files are copied to `tempdir()`, but actually all Stan files in the model directory and under `include_paths`. This shouldn't be a problem unless you have GBs of Stan files in your directory, or other invalid Stan files that cause the regex to fail.