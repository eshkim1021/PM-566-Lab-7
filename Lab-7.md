Lab 07 - Web scraping and Regular Expressions
================

``` r
knitr::opts_chunk$set(include  = TRUE)
```

# Learning goals

  - Use a real world API to make queries and process the data.
  - Use regular expressions to parse the information.
  - Practice your GitHub skills.

# Lab description

In this lab, we will be working with the [NCBI
API](https://www.ncbi.nlm.nih.gov/home/develop/api/) to make queries and
extract information using XML and regular expressions. For this lab, we
will be using the `httr`, `xml2`, and `stringr` R packages.

This markdown document should be rendered using `github_document`
document.

``` r
library(httr)
library(tidyverse)
library(xml2)
```

## Question 1: How many sars-cov-2 papers?

Build an automatic counter of sars-cov-2 papers using PubMed. You will
need to apply XPath as we did during the lecture to extract the number
of results returned by PubMed in the following web address:

    https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2

Complete the lines of code:

``` r
# Downloading the website
website <- xml2::read_html("https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2")

#Alternative 
alternative <- httr::GET(
  url = "https://pubmed.ncbi.nlm.nih.gov/",
  query = list(term = "sars-cov-2")
)

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]/span")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
stringr::str_extract(counts, "[0-9,]+")
```

    ## [1] "33,814"

Don’t forget to commit your work\!

## Question 2: Academic publications on COVID19 and Hawaii

You need to query the following The parameters passed to the query are
documented [here](https://www.ncbi.nlm.nih.gov/books/NBK25499/).

Use the function `httr::GET()` to make the following query:

1.  Baseline URL:
    <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi>

2.  Query parameters:
    
      - db: pubmed
      - term: covid19 hawaii
      - retmax: 1000

<!-- end list -->

``` r
library(httr)
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(db = "pubmed",
               term = "covid19 hawaii",
               retmax = 1000)
            )

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

The query will return an XML object, we can turn it into a character
list to analyze the text directly with `as.character()`. Another way of
processing the data could be using lists with the function
`xml2::as_list()`. We will skip the latter for now.

Take a look at the data, and continue with the next question (don’t
forget to commit and push your results to your GitHub repo\!).

## Question 3: Get details about the articles

The Ids are wrapped around text in the following way: `<Id>... id number
...</Id>`. we can use a regular expression that extract that
information. Fill out the following lines of code:

``` r
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>[0-9]+</Id>")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "<Id>|</Id>")
```

With the ids in hand, we can now try to get the abstracts of the papers.
As before, we will need to coerce the contents (results) to a list
using:

1.  Baseline url:
    <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi>

2.  Query parameters:
    
      - db: pubmed
      - id: A character with all the ids separated by comma, e.g.,
        “1232131,546464,13131”
      - retmax: 1000
      - rettype: abstract

**Pro-tip**: If you want `GET()` to take some element literal, wrap it
around `I()` (as you would do in a formula in R). For example, the text
`"123,456"` is replaced with `"123%2C456"`. If you don’t want that
behavior, you would need to do the following `I("123,456")`.

``` r
publications <- GET(
    url = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
    query = list( db = "pubmed",
                  id = paste(ids,collapse = ","),
                  retmx = 1000,
                  rettype = "abstract"
        )
    )

# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```

With this in hand, we can now analyze the data. This is also a good time
for committing and pushing your work\!

## Question 4: Distribution of universities, schools, and departments

Using the function `stringr::str_extract_all()` applied on
`publications_txt`, capture all the terms of the form:

1.  University of …
2.  … Institute of …

Write a regular expression that captures all such instances

``` r
institution <- str_extract_all(
  publications_txt,
  "University\\sof\\s[[:alpha:]]+|[[:alpha:]]+\\sInstitute\\sof\\s[[:alpha:]]+"
  ) 
institution <- unlist(institution)
table(institution)
```

    ## institution
    ##      Australian Institute of Tropical Massachusetts Institute of Technology 
    ##                                     9                                     1 
    ##   National Institute of Environmental    Prophylactic Institute of Southern 
    ##                                     3                                     2 
    ##                 University of Arizona              University of California 
    ##                                     2                                     6 
    ##                 University of Chicago                University of Colorado 
    ##                                     1                                     1 
    ##                   University of Hawai                  University of Hawaii 
    ##                                    20                                    38 
    ##                  University of Health                University of Illinois 
    ##                                     1                                     1 
    ##                    University of Iowa                University of Lausanne 
    ##                                     4                                     1 
    ##              University of Louisville                University of Nebraska 
    ##                                     1                                     5 
    ##                  University of Nevada                     University of New 
    ##                                     1                                     2 
    ##            University of Pennsylvania              University of Pittsburgh 
    ##                                    18                                     5 
    ##                 University of Science                   University of South 
    ##                                    14                                     1 
    ##                University of Southern                  University of Sydney 
    ##                                     1                                     1 
    ##                   University of Texas                     University of the 
    ##                                     5                                     1 
    ##                    University of Utah               University of Wisconsin 
    ##                                     2                                     3

Repeat the exercise and this time focus on schools and departments in
the form of

1.  School of …
2.  Department of …

And tabulate the results

``` r
schools_and_deps <- str_extract_all(
  publications_txt,
  "School\\s+of\\s+[[:alpha:]]+|Department\\sof\\s[[:alpha:]]+"
  )

unlist(schools_and_deps)
```

    ##   [1] "Department of Translational"  "Department of Experimental"  
    ##   [3] "Department of Population"     "School of Medicine"          
    ##   [5] "Department of Information"    "School of Public"            
    ##   [7] "School of Natural"            "Department of Geography"     
    ##   [9] "Department of Geography"      "School of Nursing"           
    ##  [11] "School of Medicine"           "School of Medicine"          
    ##  [13] "Department of Quantitative"   "School of Medicine"          
    ##  [15] "Department of Medicine"       "School of Medicine"          
    ##  [17] "Department of Medicine"       "Department of Psychology"    
    ##  [19] "Department of Tropical"       "Department of Psychiatry"    
    ##  [21] "Department of Psychiatry"     "Department of Psychiatry"    
    ##  [23] "School of Medicine"           "Department of Psychiatry"    
    ##  [25] "Department of Nephrology"     "School of Medicine"          
    ##  [27] "Department of Nephrology"     "School of Medicine"          
    ##  [29] "Department of Infectious"     "School of Medicine"          
    ##  [31] "Department of Nephrology"     "School of Medicine"          
    ##  [33] "Department of Infectious"     "School of Medicine"          
    ##  [35] "Department of Internal"       "School of Medicine"          
    ##  [37] "Department of Nephrology"     "School of Medicine"          
    ##  [39] "Department of Nephrology"     "School of Medicine"          
    ##  [41] "Department of Surgery"        "Department of Anesthesiology"
    ##  [43] "Department of Clinical"       "Department of Clinical"      
    ##  [45] "Department of Anesthesiology" "Department of Anesthesiology"
    ##  [47] "School of Medicine"           "Department of Obstetrics"    
    ##  [49] "Department of Obstetrics"     "School of Medicine"          
    ##  [51] "Department of Obstetrics"     "Department of Obstetrics"    
    ##  [53] "School of Medicine"           "Department of OB"            
    ##  [55] "School of Medicine"           "Department of OB"            
    ##  [57] "School of Medicine"           "Department of OB"            
    ##  [59] "School of Medicine"           "Department of OB"            
    ##  [61] "School of Medicine"           "Department of OB"            
    ##  [63] "School of Medicine"           "Department of Biology"       
    ##  [65] "Department of Biology"        "School of Public"            
    ##  [67] "School of Public"             "Department of Biology"       
    ##  [69] "School of Public"             "School of Public"            
    ##  [71] "School of Public"             "Department of Nutrition"     
    ##  [73] "Department of Family"         "School of Medicine"          
    ##  [75] "Department of Family"         "School of Medicine"          
    ##  [77] "Department of Cell"           "Department of Cell"          
    ##  [79] "Department of Cell"           "Department of Cell"          
    ##  [81] "Department of Pediatrics"     "School of Medicine"          
    ##  [83] "Department of Pediatrics"     "School of Medicine"          
    ##  [85] "Department of Pediatrics"     "Department of Pediatrics"    
    ##  [87] "Department of Pediatrics"     "Department of Pediatrics"    
    ##  [89] "Department of Medicine"       "Department of Pediatrics"    
    ##  [91] "Department of Pediatrics"     "School of Medicine"          
    ##  [93] "School of Social"             "School of Medicine"          
    ##  [95] "Department of Medicine"       "School of Medicine"          
    ##  [97] "Department of Medicine"       "School of Medicine"          
    ##  [99] "Department of Medicine"       "School of Medicine"          
    ## [101] "Department of Medicine"       "School of Medicine"          
    ## [103] "Department of Medicine"       "School of Medicine"          
    ## [105] "School of Medicine"           "School of Medicine"          
    ## [107] "School of Medicine"           "School of Medicine"          
    ## [109] "School of Medicine"           "Department of Medicine"      
    ## [111] "School of Medicine"           "School of Medicine"          
    ## [113] "School of Medicine"           "School of Medicine"          
    ## [115] "Department of Medicine"       "School of Medicine"          
    ## [117] "School of Medicine"           "Department of Surgery"       
    ## [119] "School of Medicine"           "Department of Medicine"      
    ## [121] "School of Medicine"           "Department of Native"        
    ## [123] "School of Medicine"           "Department of Native"        
    ## [125] "School of Medicine"           "Department of Tropical"      
    ## [127] "School of Medicine"           "School of Medicine"          
    ## [129] "Department of Tropical"       "School of Medicine"          
    ## [131] "School of Medicine"           "Department of Tropical"      
    ## [133] "School of Medicine"           "School of Medicine"          
    ## [135] "Department of Tropical"       "School of Medicine"          
    ## [137] "Department of Urology"        "School of Medicine"          
    ## [139] "Department of Pediatrics"     "Department of Internal"      
    ## [141] "Department of Internal"       "Department of Pediatrics"    
    ## [143] "Department of Pediatrics"     "Department of Critical"      
    ## [145] "School of Medicine"           "Department of Internal"      
    ## [147] "School of Medicine"           "Department of Pediatrics"    
    ## [149] "School of Medicine"           "Department of Internal"      
    ## [151] "School of Medicine"           "School of Medicine"          
    ## [153] "Department of Critical"       "School of Medicine"          
    ## [155] "School of Medicine"           "School of Medicine"          
    ## [157] "School of Medicine"           "Department of Otolaryngology"
    ## [159] "Department of Otolaryngology" "Department of Otolaryngology"
    ## [161] "School of Medicine"           "Department of Otolaryngology"
    ## [163] "Department of Communication"  "School of Medicine"          
    ## [165] "Department of Physical"       "School of Medicine"          
    ## [167] "Department of Rehabilitation" "Department of Veterans"      
    ## [169] "Department of Medicine"       "Department of Medicine"      
    ## [171] "Department of Medicine"       "Department of Medicine"      
    ## [173] "Department of Medicine"       "Department of Medicine"      
    ## [175] "Department of Medicine"       "Department of Medicine"      
    ## [177] "Department of Medicine"       "Department of Medicine"      
    ## [179] "Department of Medicine"       "Department of Medicine"      
    ## [181] "Department of Medicine"       "Department of Medicine"      
    ## [183] "Department of Medicine"       "Department of Medicine"      
    ## [185] "Department of Medicine"       "Department of Medicine"      
    ## [187] "Department of Medicine"       "Department of Medicine"      
    ## [189] "Department of Medicine"       "Department of Medicine"      
    ## [191] "Department of Medicine"       "Department of Medicine"      
    ## [193] "Department of Medicine"       "Department of Cardiology"    
    ## [195] "Department of Epidemiology"   "School of Public"            
    ## [197] "Department of Medical"        "Department of Nutrition"     
    ## [199] "School of Public"             "Department of Genetic"       
    ## [201] "Department of Medicine"       "Department of Medical"       
    ## [203] "Department of Preventive"     "School of Medicine"          
    ## [205] "Department of Preventive"     "Department of Computational" 
    ## [207] "Department of Medical"        "Department of Family"        
    ## [209] "Department of Epidemiology"   "School of Public"            
    ## [211] "School of Medicine"           "School of Medicine"          
    ## [213] "Department of Epidemiology"   "School of Public"            
    ## [215] "Department of Medicine"       "Department of Nutrition"     
    ## [217] "School of Public"             "Department of Epidemiology"  
    ## [219] "School of Public"             "Department of Epidemiology"  
    ## [221] "School of Public"             "Department of Medicine"      
    ## [223] "Department of Environmental"  "School of Public"            
    ## [225] "Department of Medicine"       "Department of Epidemiology"  
    ## [227] "School of Public"             "Department of Social"        
    ## [229] "School of Public"             "Department of Epidemiology"  
    ## [231] "School of Public"             "Department of Twin"          
    ## [233] "Department of Medicine"       "Department of Medicine"      
    ## [235] "Department of Medicine"       "Department of Medicine"      
    ## [237] "Department of Epidemiology"   "School of Public"            
    ## [239] "Department of Twin"           "Department of Nutrition"     
    ## [241] "School of Public"             "Department of Epidemiology"  
    ## [243] "School of Public"             "Department of Quantitative"  
    ## [245] "School of Medicine"           "Department of Quantitative"  
    ## [247] "School of Medicine"           "Department of Quantitative"  
    ## [249] "School of Medicine"           "Department of Quantitative"  
    ## [251] "School of Medicine"           "Department of Pediatrics"    
    ## [253] "School of Medicine"           "Department of Quantitative"  
    ## [255] "School of Medicine"           "School of Medicine"          
    ## [257] "Department of Surgery"        "Department of Surgery"       
    ## [259] "School of Medicine"           "Department of Surgery"       
    ## [261] "Department of Neurology"      "School of Medicine"          
    ## [263] "Department of Microbiology"   "School of Medicine"          
    ## [265] "Department of Internal"       "Department of Defense"       
    ## [267] "School of Medicine"           "Department of Anesthesiology"
    ## [269] "Department of Anesthesiology" "School of Medicine"          
    ## [271] "School of Medicine"           "Department of Physical"      
    ## [273] "School of Medicine"           "School of Medicine"          
    ## [275] "Department of Physical"       "School of Medicine"          
    ## [277] "Department of Anesthesiology" "Department of Surgery"       
    ## [279] "Department of Veterans"

``` r
table(schools_and_deps)
```

    ## schools_and_deps
    ## Department of Anesthesiology        Department of Biology 
    ##                            6                            3 
    ##     Department of Cardiology           Department of Cell 
    ##                            1                            4 
    ##       Department of Clinical  Department of Communication 
    ##                            2                            1 
    ##  Department of Computational       Department of Critical 
    ##                            1                            2 
    ##        Department of Defense  Department of Environmental 
    ##                            1                            1 
    ##   Department of Epidemiology   Department of Experimental 
    ##                            9                            1 
    ##         Department of Family        Department of Genetic 
    ##                            3                            1 
    ##      Department of Geography     Department of Infectious 
    ##                            2                            2 
    ##    Department of Information       Department of Internal 
    ##                            1                            6 
    ##        Department of Medical       Department of Medicine 
    ##                            3                           44 
    ##   Department of Microbiology         Department of Native 
    ##                            1                            2 
    ##     Department of Nephrology      Department of Neurology 
    ##                            5                            1 
    ##      Department of Nutrition             Department of OB 
    ##                            4                            5 
    ##     Department of Obstetrics Department of Otolaryngology 
    ##                            4                            4 
    ##     Department of Pediatrics       Department of Physical 
    ##                           13                            3 
    ##     Department of Population     Department of Preventive 
    ##                            1                            2 
    ##     Department of Psychiatry     Department of Psychology 
    ##                            4                            1 
    ##   Department of Quantitative Department of Rehabilitation 
    ##                            6                            1 
    ##         Department of Social        Department of Surgery 
    ##                            1                            6 
    ##  Department of Translational       Department of Tropical 
    ##                            1                            5 
    ##           Department of Twin        Department of Urology 
    ##                            2                            1 
    ##       Department of Veterans           School of Medicine 
    ##                            2                           87 
    ##            School of Natural            School of Nursing 
    ##                            1                            1 
    ##             School of Public             School of Social 
    ##                           20                            1

## Question 5: Form a database

We want to build a dataset which includes the title and the abstract of
the paper. The title of all records is enclosed by the HTML tag
`ArticleTitle`, and the abstract by `Abstract`.

Before applying the functions to extract text directly, it will help to
process the XML a bit. We will use the `xml2::xml_children()` function
to keep one element per id. This way, if a paper is missing the
abstract, or something else, we will be able to properly match PUBMED
IDS with their corresponding records.

``` r
pub_char_list <- xml2::xml_children(publications)
pub_char_list <- sapply(pub_char_list, as.character)
```

Now, extract the abstract and article title for each one of the elements
of `pub_char_list`. You can either use `sapply()` as we just did, or
simply take advantage of vectorization of `stringr::str_extract`

``` r
abstracts <- str_extract(pub_char_list, "[YOUR REGULAR EXPRESSION]")
abstracts <- str_remove_all(abstracts, "[CLEAN ALL THE HTML TAGS]")
abstracts <- str_remove_all(abstracts, "[CLEAN ALL EXTRA WHITE SPACE AND NEW LINES]")
```

How many of these don’t have an abstract? Now, the title

``` r
titles <- str_extract(pub_char_list, "[YOUR REGULAR EXPRESSION]")
titles <- str_remove_all(titles, "[CLEAN ALL THE HTML TAGS]")
```

Finally, put everything together into a single `data.frame` and use
`knitr::kable` to print the results

``` r
database <- data.frame(
  "[DATA TO CONCATENATE]"
)
knitr::kable(database)
```

Done\! Knit the document, commit, and push.

## Final Pro Tip (optional)

You can still share the HTML document on github. You can include a link
in your `README.md` file as the following:

``` md
View [here](https://ghcdn.rawgit.org/:user/:repo/:tag/:file)
```

For example, if we wanted to add a direct link the HTML page of lecture
7, we could do something like the following:

``` md
View [here](https://ghcdn.rawgit.org/USCbiostats/PM566/master/static/slides/07-apis-regex/slides.html)
```
