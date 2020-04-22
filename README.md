# Search Engine
Project in [ Language Engineering ] FCS Level 3

---

Install Libs
```
pip install b4
nltk.download('punkt')
#nltk.download('wordnet')
#nltk.download('stopwords')
```
###### Libraries Used
```
import requests 
from bs4 import BeautifulSoup
import re
import nltk
from nltk.corpus import wordnet 
from nltk.corpus import stopwords
```
Search by using google query
```
query = input("Enter word to search: ")
url = f"https://google.com/search?q={query}" 
res = requests.get(url, headers={"User-Agent":"Mozilla/0.5"})
```
Extract HTML Content and all divs that contain title and description , link by class of div
```
soup = BeautifulSoup(res.content, "html.parser")
result_div = soup.find_all('div', attrs = {'class': 'ZINbbc'})
```
```
links = [ ] 		# Store Links 
titles = [ ]		# Store Titles
descriptions = [ ]	# Store  Descriptions
```
```
for r in result_div:
    try:
        link = r.find('a', href = True)
        title = r.find('div', attrs={'class':'vvjwJb'}).get_text()
        description = r.find('div', attrs={'class':'s3v9rd'}).get_text()
		# Check if extract empty data else store to arrays
        if link != '' and title != '' and description != '': 
            links.append(link['href'])
            titles.append(title)
            descriptions.append(description)
    except:
        continue
```
```
to_remove = [ ]
clean_links = [ ]
```
```
for i, l in enumerate(links):		# enumerate return index and value
	# make sure that file is a link by start with /url\?q
    clean = re.search('\/url\?q\=(.*)\&sa',l)
    # if return none value means not a useful link 
    if clean is None:
        to_remove.append(i)
        continue
    clean_links.append(clean.group(1))

# remove title and descript of none links from arrays
for x in to_remove:
    del titles[x]
    del descriptions[x]
```
function sort tuple by second value or any determine by ind
```
def Sort_Tuple(tup,ind):
    tup.sort(key = lambda x: x[ind])  
    return tup  
```
```
List_all_rank = [ ] 	# store rank by page
def lesk(query, sentence,ind):
    Text1 = sentence.lower()	# string to lowercase
    words = nltk.word_tokenize(Text1)
    
    stop_words = stopwords.words("english")
    stop_words += ['can', 'will', 'use', 'one', 'using', 'used', 'also', 'see', 'first', 'like']
    stop_words += ['page', 'get', 'new', 'two', 'site', 'blog', 'many', 'may' ,"don't", 'dont', 'way']
    stop_words += ['last', 'best', 'able', 'even', 'next', 'last', 'let', "none", 'every', 'three']
    stop_words += ['lot', 'well', 'chart', 'much', 'based', 'important', 'posts', 'reads', 'least']
    stop_words += ['still', 'follow', 'called', 'and','this', 'that', 'there', 'as','the', 'is']
    stop_words += ['/', '=', '.', ',', '.']

    filtered_words = []
    for word in words:
        if word not in stop_words:
            filtered_words.append(word)
    word = query
    len_syn = len(wordnet.synsets(word))	# length all synsets
    
    da=[ ]	# store definition to avoid repetition
    ea=[ ]	# store examples to avoid repetition
    List_rank = [ ] 	# store rank by synsets
	# How many words description is mentioned in each synset
	# at the end get max and store in List_all_rank = [ ]

    for x in range(len_syn):

        for i in filtered_words:
            synset = wordnet.synsets(word)[x]
            if i in synset.definition().lower():
                if i in da:
                    pass
                else:
                    da.append(i)
              
            if len(synset.examples()) == 0:
                pass
            else:
                if i in synset.examples()[0].lower() and word in synset.examples()[0].lower():
                    if i in ea:
                        pass
                    else:
                        ea.append(i)
                        
		# create array and add first item number of synset
        l = []
        l.insert(0,x)
        r = len(da)+len(ea) # collection of description word found in each synset
        l.append(r)
        y2 = tuple(l) # convert list to tuple
        List_rank.append(y2)
    Sorted_list_rank = Sort_Tuple(List_rank,1) #sort by second item [number of words]
    l = list(Sorted_list_rank[-1]) # after sort get last tuple ,Express the greatest value then convert to list
    l.insert(0,ind) # add first item number of page
    y2 = tuple(l) 	# convert tuple again
    List_all_rank.append(y2)
    
for idx, val in enumerate(descriptions):
    lesk(query, val, idx)

s_li = Sort_Tuple(List_all_rank,2) # (numOf page, numOf syn, how many words) here sort by how many words 
for x in range(len(List_all_rank),0,-1): 
    print("Page Number",s_li[x-1][0]+1, ", Most Synset Num",s_li[x-1][1],", With Rank",s_li[x-1][2])
    print("Title:",titles[s_li[x-1][0]])
    print("Link:",clean_links[s_li[x-1][0]])
    print("-------------------------\n")
```
