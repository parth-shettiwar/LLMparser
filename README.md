# LLMparser
The purpose of this code is to write an efficient parser for specifically Law documents. 
The complete code is written in parse.ipynb
Following libraries have been used. Please install them running the notebook. 
1) pandas
2) fitz
3) re
4) json
5) os
6) opeanai
7) fuzzywuzzy

Overal structure of code
The code mainly has following flow for each document:
1) Finding contender pages for TOC using openai api (function: toc_detector)
2) A robust function (function: toc_extractor) is written to parse the toc from these pages and populated in a dataframe
3) This is followed by function to find the actual pages where each section header is (The page number from toc is not the actual page of pdf, say when opened in a pdf reader).
4) Once we find actual pages for each section, a function is written to populate the starting and ending indices (on word level) on actual page for each section header from toc. Fuzzy match (function: fuzzy_search_page) is used for this and pythons inbuilt library fuzzywuzzy is used for this
5) Finally we populate the text between current header ending index and next header starting index as the section body for the current header

Some design parameters:
1) tolerance: set to 1 currently. This is part of internal implementation when we use openai api to classify each page as toc or not and this determines the stopping criteria. If we get more than tolerance pages of "Not a toc page", then we stop
2) min_toc_bound, max_toc_bound: assumption that toc will come between min_toc_bound and max_toc_bound
3) word_threshold: max length of standard header 
4) fuzzy match min threshold: This is currently set to 80 in the fuzzy_search_page function. This parameter is to decide when to consider that we have found the header in the document with good accuracy. Max value can be 100. We should have a match with a document header with atleast this much threshold

Notes on some functions:
1) fuzzy_search_page: This has 2 parts, once I fuzzy match with a line to line basis only and then if it fails, I go to do perfect match as taking header first and finding same text in complete page text. The line to line basis is important, since words like summary, which are usually headers, also happen to be occuring many times in a page as non-headers embedded in paragraphs. Hence perfect match match wont work, since it will give lot of false positivefor indices.
2) The toc_detector function is just to get potential candidates for toc pages (After  trying multiple prompts, this function is also made quite robust in terms of not missing any toc page). As its said, a determinstic layer should always be put over ML based operation. Hence toc_extractor filters the correct pages from toc pages candidates. The function is made quite robust to handle any inconsistencies like header and footers and any other content which can pop up on toc page. 
3) PyMuPDf can be used to remove headers and footers from a page, using rectangular box crop. However, this suffers from many false positives cases
4) Some documents have some toc and then directly the actual text starts from the same page. Such cases are handled differently by saving the final word index of toc on a page. 
