The complete code is written in parse.ipynb
Just upload the notebook to VSCode or any code editor which supports Jupyter Notebook
Following libraries have been used. Please install them running the notebook. I have written the corresponding pip command for installation
1) pandas: pip install pandas
2) fitz: pip install fitz
3) re: pip install regex
4) json: pip install jsonlib
5) os: pip install os-sys
6) opeanai: pip install openai
7) fuzzywuzzy:pip install fuzzywuzzy

Notes on results:
1) The document CITRIX SYSTEMS INC_20220316_DEFM14A_19951567_4414270.Pdf is not there in final chunk. I also checked manually, the document does not
contain a table of contents, hence section header and body could not be extracted for this document
2) In final chunks, some csvs contain headers which dont have any section body to them. With manual verification, this is the intended behaviour since sometimes the headers are structured where a particular header follows immediately after a previous header. This leads to no section body for previous header
3) Due to time constraints, only mandatory part is given in final results.(Since generating results for all sections of all documents was taking time)
An approach is tried for finding the class for each section. 
Using online sources and gpt, I generated a base prompt where I write some key words and phrases used for each of the three categpries in general. With this base prompt, I prompted gpt to classify what class a particular (header + section body) belongs to. I have ran the result for 1 document and attached in notebook. Due to token limimt size of 8192 tokens, have curbed the section body to 4000 words. Written function: classify_section

Overal structure of code for mandatory part
The code mainly has following flow for each document:
1) Finding contender pages for TOC using openai api (function: toc_detector)
2) A robust function (function: toc_extractor) is written to parse the toc from these pages and populated in a dataframe
3) This is followed by function to find the actual pages where each section header is (The page number from toc is not the actual page of pdf, say when opened in a pdf reader).
4) Once we find actual pages for each section, a function is written to populate the starting and ending indices (on word level) on actual page for each section header from toc. Fuzzy match (function: fuzzy_search_page) is used for this and pythons inbuilt library fuzzywuzzy is used for this
5) Finally we populate the text between current header ending index and next header starting index as the section body for the current header

Notes:
1) Currently some documents contains 2 TOCs, where second TOC is in Annexure. This functionality is currently not handled as part of this code, however can be easily expanded by following the same pipeline above. Due to time constraints, this is not done
2) For a given TOC, I am currently extracting all sections with page numbers and roman numerals. This can be further expanded to handle different types of page number formats. Hence Annexure is not being populated currently in the toc. This is based on premise that for analysing a proxy document, annexure is less important.

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
