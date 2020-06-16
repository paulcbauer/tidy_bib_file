
<!-- README.md is generated from README.Rmd. Please edit that file -->

# tidy\_bib\_file

`tidy_bib_file` can be used to tidy up (delete unused references from) a
larger bib file based on the citations in a .rmd file. It creates a new
cleaned bib file. Depending on how “clean” the original bib file is the
function may throw errors. Normally, R will tell you in which line of
your bib file the error is once you compile paper.rmd.

## Installation

The function `tidy_bib_file` is currently not part of a package. So
simply run the code below to have it available. You will need the
`df2bib` package.

``` r
# FUNCTION ####
tidy_bib_file <- function(rmarkdownfile,
                          old_bib_file,
                          new_bib_file,
                          repair = FALSE,
                          replace_curly_braces = FALSE){

library(stringr)
library(bib2df)
library(dplyr)

# Read in rmd file as one long string
  text <- paste(readLines(rmarkdownfile),collapse=" ")

# Extract citation keys
  citation_keys1 <- gsub("]$", "", unlist(regmatches(text, gregexpr("@[-A-Za-z0-9_]*\\]", text))))
  citation_keys2 <- gsub("\\s$", "", unlist(regmatches(text, gregexpr("@[-A-Za-z0-9_]*\\s", text))))
  citation_keys3 <- gsub("\\;$", "", unlist(regmatches(text, gregexpr("@[-A-Za-z0-9_]*;", text))))
  citation_keys4 <- gsub("\\,$", "", unlist(regmatches(text, gregexpr("@[-A-Za-z0-9_]*,", text))))

# Merge citation keys and keep unique
  citation_keys <- gsub("@", "", unique(c(citation_keys1, citation_keys2, citation_keys3, citation_keys4)))
  citation_keys <- data.frame(BIBTEXKEY = as.character(citation_keys))
  
  cat("Finished extracting bibtexkeys.\n")
  
  
# Read bib file #### 
  df <- bib2df(old_bib_file)
  cat("Imported old bib file.\n")
  
  bibliography_new <- inner_join(df, citation_keys, by = "BIBTEXKEY")

# Clean title
  bibliography_new$TITLE <- str_replace_all(bibliography_new$TITLE,
                                            "\\}|\\{", "")
  bibliography_new$TITLE <- str_to_title(bibliography_new$TITLE)
  
# Create new bib file ####
  df2bib(bibliography_new,  new_bib_file)
  
  
  cat("Created new bib file.\n")
  
# Repair bib file ###
  if(isTRUE(repair)){
    bibfile <- readLines(new_bib_file, encoding = "UTF-8") 
    accents <- c("\\`", "\\'", "\\^", '\\"', "\\~", "\\=", 
                 "\\.", "\\u", "\\v", "\\H", "\\t", "\\c", "\\d", "\\b", "\\k")
    accentletters <- expand.grid(accents, letters)
    accentletters <- sprintf('%s%s', accentletters[,1], accentletters[,2])
    accentletters <- gsub("\\\\", "", accentletters)
    
    # for(i in accentletters){bibfile <- gsub("\\{\\\\i\\}", "\\{\\\\i\\}\\}", bibfile)}
    bibfile <- gsub("\\{\\\\'e\\}", "\\{\\\\'e\\}\\}", bibfile)
    bibfile <- gsub("\\{\\\\'a\\}", "\\{\\\\'a\\}\\}", bibfile)
    bibfile <- gsub("\\{\\\\'o\\}", "\\{\\\\'o\\}\\}", bibfile)
    
    writeLines(bibfile, new_bib_file)
    
    cat("Repaired new bib file.\n")
  }
  

  
# Replace curly braces
  if(isTRUE(replace_curly_braces)){
  bibfile <- readLines(new_bib_file, encoding = "UTF-8") 
  bibfile <- str_replace_all(bibfile, "\\s\\{", ' "')
  bibfile[nchar(bibfile)>1] <- str_replace_all(bibfile[nchar(bibfile)>1], 
                                                "\\}", '"')
  #bibfile <- bibfile[!bibfile==""]
  writeLines(bibfile, new_bib_file)
  
  cat("Also replaced curly braces.\n")
  }
}
```

## Example: Single bib file

See below for how to create a cleaned bib file for your paper. Make sure
to download the files into your working directory.

``` r
tidy_bib_file(rmarkdownfile = "paper.Rmd", 
              old_bib_file = "bibliography_all.bib",
              new_bib_file = "bibliography_paper.bib",
              repair = FALSE,
              replace_curly_braces = FALSE)
```

    ## Finished extracting bibtexkeys.

    ## Warning: `as_data_frame()` is deprecated as of tibble 2.0.0.
    ## Please use `as_tibble()` instead.
    ## The signature and semantics have changed, see `?as_tibble`.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_warnings()` to see where this warning was generated.

    ## Imported old bib file.
    ## Created new bib file.

## Example: Loop over bib files

See below for how to create a single bibligraphy from several bib files.
Subsequently you can subset the bib file again.

``` r
# LOOP over single bib files ####
  bib_complete <- NULL
  files <- c("references1.bib", "references2.bib", "references3.bib", "references4.bib")
  
  for(i in files){
    bib_i <- bib2df(i)
    bib_complete <- bind_rows(bib_complete, bib_i)
    # Delete irrelevant categories (that may contain errors)
    bib_complete <- bib_complete %>% dplyr::select(-contains("ABSTRACT"),
                                                   -contains("MENDELEY.TAGS"),
                                                   -contains("HYPOTHESIZED"))
  }
```

    ## Warning in readLines(file): incomplete final line found on 'references4.bib'

``` r
  # Save bib file
  df2bib(bib_complete, "bibliography_all2.bib")
  
# Use new .bib file to create bibliography for paper
  tidy_bib_file(rmarkdownfile = "paper.Rmd", 
                old_bib_file = "bibliography_all2.bib",
                new_bib_file = "bibliography_paper.bib",
                repair = FALSE,
                replace_curly_braces = FALSE)  
```

    ## Finished extracting bibtexkeys.
    ## Imported old bib file.
    ## Created new bib file.
