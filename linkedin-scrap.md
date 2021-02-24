---
description: beautifulsoup selenium bot webscraping automation linkedin chrome webbrowser
---

# LinkedIn Scrap

The following code creates a bot with the aim of extracting data from a series of LinkedIn users, on the basis of given parameters.

The bot has two tasks: 

* it opens Chrome, types the input string in the search bar and downloads locally the html source code from the first list of searched users
* it analyzes the html code in order to extract, for each user, these pieces of information: name, occupation, location, personal link.

## Output and Full Code​

![](.gitbook/assets/immagine1%20%281%29.png)

```python
import warnings
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from bs4 import BeautifulSoup
import datetime
import time
import pandas as pd
from tinydb import TinyDB, Query
import io
import random
import re
import traceback
import numpy as np
import os
import codecs
import sys
import tqdm
import docx


#configuration
user = ''
pwd = ''

path_to_chromedriver = ''
local_path = ''

firms_to_search = ['Credem', 'Intesa San Paolo']
titles = ['CIO', 'Head of', 'Innovation']


#handlers
def login(path_to_chromedriver, user_name, user_psw):
    usr, psw = user_name, user_psw
    url = 'https://www.linkedin.com/uas/login?_l=it'
    browser = webdriver.Chrome(executable_path=path_to_chromedriver)
    browser.get(url)
    Box1 = browser.find_element_by_name('session_key')
    Box1.send_keys(usr)
    Box2 = browser.find_element_by_name('session_password')
    Box2.send_keys(psw)
    Box2.send_keys(Keys.RETURN)
    browser.maximize_window()
    return browser
    
def roll_dices(minn, maxx):
    return random.randint(minn, maxx)


def scroll(browser):
    for i in range(1, 6, 1):
        browser.execute_script("window.scrollTo(0, %s);" % (800 * i))
        time.sleep(roll_dices(2, 4))


def get_page_html(browser, html_path, firm):
    scroll(browser)
    buttons = browser.find_elements_by_tag_name('button')
    for elem in buttons:
        if 'Mostra' in elem.text:
            try:
                elem.click()
            except:
                pass
            time.sleep(roll_dices(4, 7))
    with io.open("Def/%s/%s.html" % (html_path, firm), "w", encoding="utf-8") as f:
        f.write(browser.page_source)


def get_soup(browser):
    html_source = browser.page_source
    soup = BeautifulSoup(html_source, "html.parser")
    return soup


def wait(time_to_wait):
    for i in range(0, time_to_wait, 1):
        print(time_to_wait - i, end=',')
        time.sleep(1)
    print('0')


def search_people(browser, searching_content):
    search_bar = browser.find_element_by_id('global-nav-typeahead')
    search_bar = search_bar.find_element_by_tag_name('input')
    print(search_bar)
    search_bar.send_keys(searching_content)
    search_bar.send_keys(Keys.RETURN)
    wait(2)
    get_page_html(browser, t, c)
    search_bar.clear()




#main functions

#Part One: getting searched pages' html locally
def html_extraction(path_to_chromedriver, user, pwd, firms_to_search, titles)
    browser = login(path_to_chromedriver, user, pwd)
    
    for f in firms_to_search:
        for t in titles:
            search_people(browser, f + ' ' + t)




#Part Two: extract from html the information
#single piece of informations' functions
def name(soup):
    name = soup.find('span', class_="name actor-name").get_text()
    return name

def role_firm(soup):
    role_firm = soup.find('p', class_="subline-level-1 t-14 t-black t-normal search-result__truncate").get_text()
    return role_firm

def location(soup):
    location = soup.find('p', class_="subline-level-2 t-12 t-black--light t-normal search-result__truncate").get_text()
    return location

def href(soup):
    href = soup.find('a', class_="search-result__result-link ember-view").get('href')
    return href


#extraction
def massive_data_acquisition(titolo, azienda):
    working_dir = os.path.join(local_path, titolo)
    try:    
        html_text=codecs.open(working_dir + '\%s.html' %azienda, 'r', 'utf-8').read()
        print(working_dir + '\%s.html' %azienda)
        soup = BeautifulSoup(html_text, 'html.parser')
        n = 1
        for elem in soup.find_all('div', class_="search-result__wrapper"):
            searched_firm.append(str(azienda))
            try:
                name.append(name(elem))
            except:
                name.append(np.nan)
            try:
                role.append(role_firm(elem))
            except:
                role.append(np.nan)
            try:
                place.append(location(elem))
            except:
                place.append(np.nan)
            try:
                link.append('https://www.linkedin.com' + href(elem))
            except:
                link.append(np.nan)
            ranking.append(n)
            n = n + 1
    except:
        searched_firm.append(str(azienda))
        name.append(np.nan)
        role.append(np.nan)
        place.append(np.nan)
        link.append(np.nan)
        ranking.append(np.nan)



#main
def main():
    
    html_extraction(path_to_chromedriver, user, pwd, firms_to_search, titles)
    
    # Storing data
    name = []
    role = []
    place = []
    searched_firm = []
    link = []
    ranking = []
    
    for t in titles:
        for f in firms_to_search:
            try:
                massive_data_acquisition(t, f)
            except:
                print('Not able to find {} data'.format(c))   
    
    
    data_dict = {'Azienda cercata': searched_firm, 'Ranking ricerca': ranking, 
                'Nome': name, 'Ruolo': role, 'Provenienza': place, 
                'Link': link}
                
    df = pd.DataFrame(data_dict)
    
    
    
    # Cleaning the output
    df['Provenienza'] = df['Provenienza'].replace(r'\r\n', '', regex=True)
    df['Provenienza'] = df['Provenienza'].replace(r'\r\n ', '', regex=True)
    df['Provenienza'] = df['Provenienza'].replace('    ', '', regex=True).replace('  ','', regex=True)
    df['Ruolo'] = df['Ruolo'].replace(r'\r\n', '', regex=True)
    df['Ruolo'] = df['Ruolo'].replace('        ', '', regex=True).replace('    ','', regex=True)
    
    
    #Export the output in csv
    df.to_csv('bot_linekdin_results.csv', index=False)
    


if __name__ == "__main__":
    main()
```

## Details

Some detail here...

### Configuration

Configuration of the personal parameters to access LinkedIn, to run the BOT \(Chromedriver\) and store local data and, lastly, to make the research \(i.e. the BOT will type in the searching bar the name of the firm and the titles, given a single couple of firm and title.

```python
user = ''
pwd = ''

path_to_chromedriver = ''
local_path = 

firms_to_search = ['Credem', 'Intesa San Paolo']
titles = ['CIO', 'Head of', 'Innovation']
```

#### 

### Handlers

Set the common handlers. Here there's a list of function that are called during the bot action. The most remarkable ones are `search_people(browser, searching_content)` which isolates the searching bar, clears it and types into it the input text, and`get_page_html(browser, html_path, firm)` which scrolls the entire page and save the whole html code in the defined path.

```python
def login(path_to_chromedriver, user_name, user_psw):
    usr, psw = user_name, user_psw
    url = 'https://www.linkedin.com/uas/login?_l=it'
    browser = webdriver.Chrome(executable_path=path_to_chromedriver)
    browser.get(url)
    Box1 = browser.find_element_by_name('session_key')
    Box1.send_keys(usr)
    Box2 = browser.find_element_by_name('session_password')
    Box2.send_keys(psw)
    Box2.send_keys(Keys.RETURN)
    browser.maximize_window()
    return browser
    
def roll_dices(minn, maxx):
    return random.randint(minn, maxx)


def scroll(browser):
    for i in range(1, 6, 1):
        browser.execute_script("window.scrollTo(0, %s);" % (800 * i))
        time.sleep(roll_dices(2, 4))


def get_page_html(browser, html_path, firm):
    scroll(browser)
    buttons = browser.find_elements_by_tag_name('button')
    for elem in buttons:
        if 'Mostra' in elem.text:
            try:
                elem.click()
            except:
                pass
            time.sleep(roll_dices(4, 7))
    with io.open("Def/%s/%s.html" % (html_path, firm), "w", encoding="utf-8") as f:
        f.write(browser.page_source)


def get_soup(browser):
    html_source = browser.page_source
    soup = BeautifulSoup(html_source, "html.parser")
    return soup


def wait(time_to_wait):
    for i in range(0, time_to_wait, 1):
        print(time_to_wait - i, end=',')
        time.sleep(1)
    print('0')


def search_people(browser, searching_content):
    search_bar = browser.find_element_by_id('global-nav-typeahead')
    search_bar = search_bar.find_element_by_tag_name('input')
    print(search_bar)
    search_bar.send_keys(searching_content)
    search_bar.send_keys(Keys.RETURN)
    wait(2)
    get_page_html(browser, t, c)
    search_bar.clear()
```

#### 

### Principal functions

The first main function gets the html codes locally, iterating on the above functions

```python
def html_extraction(path_to_chromedriver, user, pwd, firms_to_search, titles)
    browser = login(path_to_chromedriver, user, pwd)
    
    for f in firms_to_search:
        for t in titles:
            search_people(browser, f + ' ' + t)
```

The second group of functions extracts the required data from the html code

```python
def name(soup):
    name = soup.find('span', class_="name actor-name").get_text()
    return name

def role_firm(soup):
    role_firm = soup.find('p', class_="subline-level-1 t-14 t-black t-normal search-result__truncate").get_text()
    return role_firm

def location(soup):
    location = soup.find('p', class_="subline-level-2 t-12 t-black--light t-normal search-result__truncate").get_text()
    return location

def href(soup):
    href = soup.find('a', class_="search-result__result-link ember-view").get('href')
    return href
```



The last principal function generates the output. By running the above functions, the ouptut is organized into a lists, containing, role, profile link, firm searched in the bar and the ranking in the research.

```python
def massive_data_acquisition(titolo, azienda):
    working_dir = os.path.join(local_path, titolo)
    try:    
        html_text=codecs.open(working_dir + '\%s.html' %azienda, 'r', 'utf-8').read()
        print(working_dir + '\%s.html' %azienda)
        soup = BeautifulSoup(html_text, 'html.parser')
        n = 1
        for elem in soup.find_all('div', class_="search-result__wrapper"):
            searched_firm.append(str(azienda))
            try:
                name.append(name(elem))
            except:
                name.append(np.nan)
            try:
                role.append(role_firm(elem))
            except:
                role.append(np.nan)
            try:
                place.append(location(elem))
            except:
                place.append(np.nan)
            try:
                link.append('https://www.linkedin.com' + href(elem))
            except:
                link.append(np.nan)
            ranking.append(n)
            n = n + 1
    except:
        searched_firm.append(str(azienda))
        name.append(np.nan)
        role.append(np.nan)
        place.append(np.nan)
        link.append(np.nan)
        ranking.append(np.nan)
```

\*\*\*\*

### Main

**Step 5**. Definition of the `main()` function and data cleaning. In the main, the whole process is called and the lists are inserted into a dictionary which generates a DataFrame. Therefore, the text cells are cleaned by spaces and new line commands.

```python
def main():
    
    html_extraction(path_to_chromedriver, user, pwd, firms_to_search, titles)
    
    # Storing data
    name = []
    role = []
    place = []
    searched_firm = []
    link = []
    ranking = []
    
    for t in titles:
        for f in firms_to_search:
            try:
                massive_data_acquisition(t, f)
            except:
                print('Not able to find {} data'.format(c))   
    
    
    data_dict = {'Azienda cercata': searched_firm, 'Ranking ricerca': ranking, 
                'Nome': name, 'Ruolo': role, 'Provenienza': place, 
                'Link': link}
                
    df = pd.DataFrame(data_dict)
    
    
    
    # Cleaning the output
    df['Provenienza'] = df['Provenienza'].replace(r'\r\n', '', regex=True)
    df['Provenienza'] = df['Provenienza'].replace(r'\r\n ', '', regex=True)
    df['Provenienza'] = df['Provenienza'].replace('    ', '', regex=True).replace('  ','', regex=True)
    df['Ruolo'] = df['Ruolo'].replace(r'\r\n', '', regex=True)
    df['Ruolo'] = df['Ruolo'].replace('        ', '', regex=True).replace('    ','', regex=True)
    
    
    #Export the output in csv
    df.to_csv('bot_linekdin_results.csv', index=False)
```

## Final thoughts / recommendation

Use this at your own risk!

Created by: Francesco Carelli and Luca Boschini

