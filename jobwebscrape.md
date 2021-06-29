# Job Site Webscraping application

<img src="images/jobcover.png?raw=true" style="width:500px;"/><br>

**Project description:** I wrote a script using Python and the Beautifulsoup library. The script parses through the pages of the website and searches for the user's input keyword on the jobtitle as well as the number of pages to run through. The script also exports the information into a .csv file. 
Due to legal reasons, this is just for education purposes and for my personal use to understand the job market for a specific position (Complying to the website's term of use).

<br>
<br>
<br>

## 1. Imports

```python
from bs4 import BeautifulSoup
import requests
import time
import re
import os
import csv
```
<br>

The above are libraries are imported:
1. Beautiful Soup: Beautiful Soup is a Python package for parsing HTML and XML documents. It generates a parse tree for parsed pages that can be used to extract data from HTML.
2. Requests: For making HTTP requests in Python.
3. Time: Used for setting a delay between interation.
4. Re: For enabling Regex to be used (e.g. .split, findall, etc.)
5. OS: For directory pathing where the .csv is generated and saved.
6. CSV: For generating, appending .CSV files

<br>

## 2. .CSV Exporting Function

```python
with open ('C:/Users/Username/data_analyst.csv, 'a', encoding='utf-8', newline='') as f_output:
    csv_print = csv.writer(f_output)

    file_is_empty = os.stat ('C:/Users/Username/data_analyst.csv').st_size == 0
    if file_is_empty:
        csv_print.writerow(['Job Title', 'Company', 'Location', 'Salary', 'Date', 'Link'])
```
<br>

I followed an online tutorial to understand and also get the base form of the above script. <a href="https://www.youtube.com/watch?v=exf14s7xJeE&ab_channel=PropagateKnowledge">Tutorial Link</a>. <br>In summary, what this script does is it opens any existing .CSV file specific to the title that was given, appends the new data onto new lines / rows. If the file is empty, it creates the header row such as Job Title, Company ,etc.

<br>

## 3. User Input and For Loop Iteration

```python
inputpage = int(input("Input Page no.: "))

pagenumbersconvert = range(1,(inputpage+1),1)
pagenumberlistconvert =[*pagenumbersconvert]
pages = pagenumberlistconvert

inputkeyword = input("Input Keyword search: ")
inputkeywordconvert = inputkeyword.replace(' ', '-')

with open ('C:/Users/Username/data_analyst.csv, 'a', encoding='utf-8', newline='') as f_output:
    csv_print = csv.writer(f_output)

    file_is_empty = os.stat ('C:/Users/Username/data_analyst.csv').st_size == 0
    if file_is_empty:
        csv_print.writerow(['Job Title', 'Company', 'Location', 'Salary', 'Date', 'Link'])


    for page in pages:
        source = requests.get('https://www.JobWebsiteExample.com.sg/en/job-search/{}-jobs/{}/'.format(inputkeywordconvert,page)).text

        soup = BeautifulSoup(source,'lxml')
```
<br>

The next part was creating a user input function to allow the users to input the keyword that they wanted to search (for e.g Data Anaylst) and the number of pages that they would like to search up to. This was done through Python's built-in Input and a for loop.

Example image below of the user terminal input:

<br>

<img src="images/UserTerminalInput.png?raw=true" style="width:500px;"/><br>

<br>

## 4. Beautiful Soup Extraction

```python
    for page in pages:
        source = requests.get('https://www.JobWebsiteExample.com.sg/en/job-search/{}-jobs/{}/'.format(inputkeywordconvert,page)).text

        soup = BeautifulSoup(source,'lxml')

        for jobs in soup.find_all('div' ,class_='someclass):
                try:
                    job_title = jobs.find('div', class_='someclass1').text.strip()
                except Exception as e:
                    job_title = None
                print('Job Title:', job_title)

                try:
                    company = jobs.find('span', class_='someclass2').text.strip()
                except Exception as e:
                    company = "Company Confidential"
                print('Company:', company)

                try:
                    location = jobs.find('span',class_='someclass3').text.strip()
                except Exception as e:
                    location = None
                print('Location:', location)

                salary = [a.text.strip() for a in jobs.select("someclass4 > div:nth-child(1) > span:nth-child(4)")]
                salary1 = str(salary).replace("[","").replace("]","").replace("'","").replace(r"\xa0"," ")
                print('Salary:', salary1)

                try:
                    date = jobs.find('time').text.strip()
                except Exception as e:
                    date = None
                print('Date Posted:', date)

                link = jobs.find('a', href=True)
                links = str(link['href'])
                linkfinal = str('https://www.JobWebsiteExample.com.sg') + links
                print('URL:', linkfinal)

                csv_print.writerow([job_title, company, location, salary1, date, linkfinal])

                print('-----------------------------')
```
<br>

Based on our the headers ('Job Title', 'Company', 'Location', 'Salary', 'Date', 'Link'), these are the information that we want to be printed and saved into our .CSV file. We use the Beautifulsoup .findall to access and find the main container class which houses the information of each Job entry on the browser webpage. Example below:

<br>

<img src="images/containerentry.png?raw=true" style="width:500px;"/><br>

<br>

Then for each information which we are interested in (e.g. Job Title), we find the associated class which contains the string text of the information we want and apply .text.strip() to get the text and remove any leading and trailing whitespaces.<br>
We repeat this for all the information. (*note: for salary we use .select instead as there are multiple main and child classes using the same name*)

Example image below shows the information extracted from the browser webpage:

<br>

<img src="images/Examplebrowserprint.png?raw=true" style="width:500px;"/><br>

<br>

## 5. Full Script

```python
from bs4 import BeautifulSoup
import requests
import time
import re
import os
import csv

inputpage = int(input("Input Page no.: "))

pagenumbersconvert = range(1,(inputpage+1),1)
pagenumberlistconvert =[*pagenumbersconvert]
pages = pagenumberlistconvert

inputkeyword = input("Input Keyword search: ")
inputkeywordconvert = inputkeyword.replace(' ', '-')


with open ('C:/Users/Username/data_analyst.csv', 'a', encoding='utf-8', newline='') as f_output:
    csv_print = csv.writer(f_output)

    file_is_empty = os.stat ('C:/Users/Username/data_analyst.csv').st_size == 0
    if file_is_empty:
        csv_print.writerow(['Job Title', 'Company', 'Location', 'Salary', 'Date', 'Link'])


    for page in pages:
        source = requests.get('https://www.JobWebsiteExample.com.sg/en/job-search/{}-jobs/{}/'.format(inputkeywordconvert,page)).text

        soup = BeautifulSoup(source,'lxml')

        for jobs in soup.find_all('div' ,class_='someclass):
                try:
                    job_title = jobs.find('div', class_='someclass1').text.strip()
                except Exception as e:
                    job_title = None
                print('Job Title:', job_title)

                try:
                    company = jobs.find('span', class_='someclass2').text.strip()
                except Exception as e:
                    company = "Company Confidential"
                print('Company:', company)

                try:
                    location = jobs.find('span',class_='someclass3').text.strip()
                except Exception as e:
                    location = None
                print('Location:', location)

                salary = [a.text.strip() for a in jobs.select("someclass4 > div:nth-child(1) > span:nth-child(4)")]
                salary1 = str(salary).replace("[","").replace("]","").replace("'","").replace(r"\xa0"," ")
                print('Salary:', salary1)


                try:
                    date = jobs.find('time').text.strip()
                except Exception as e:
                    date = None
                print('Date Posted:', date)

                link = jobs.find('a', href=True)
                links = str(link['href'])
                linkfinal = str('https://www.JobWebsiteExample.com.sg') + links
                print('URL:', linkfinal)

                csv_print.writerow([job_title, company, location, salary1, date, linkfinal])

                print('-----------------------------')

                
    time.sleep(0.5)
```

*END*
