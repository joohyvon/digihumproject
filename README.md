# Digihum Project
## Guggenheim in Suomi24
This is a project done for the *Introduction to methods in digital humanities* course. I was interested in trying to map the kind of context in which the planned Guggenheim Museum in Helsinki was being discussed among the so called *kansan syvät rivit*. Being an artist myself, I've noted in my social media feeds a need at times, often in relation to the Guggenheim project, for art talk and practice more integrated or embedded to the social. The Guggenheim Helsinki project has also sprung up some reactive collectives/organizations like the artist run Checkpoint Helsinki. I wanted to see whether visual arts get featured in the discussions around Guggenheim on Suomi24, or if the matter is seen in terms of economics and politics outside of the *so-to-say* artworld. 

### What I did
I used the Suomi24 discussion forum data from the [Korp API](https://www.kielipankki.fi/support/korpapi/ "Title"). The collection contains the discussions between 1.1.2001 and 24.9.2016, so I do miss the latest turns in the saga. Similar to the practice during the course, I used Python to get the search results altering the code just a bit for my purposes, quering for gugg.* to get all the different nicknames given to Guggenheim very early on (this gave some unrelated hits as well, but they seem mostly to feature before the Guggenheim project and not heavy enough to really effect the results). Here's the code for Python

```python
corpus = "s24_001,s24_002,s24_003,s24_004,s24_005,s24_006,s24_007,s24_008,s24_009,s24_010"
corpus = corpus.upper()
query = '(gugg).*'
end = '2000'
```
```python
import requests

response = requests.get("https://korp.csc.fi/cgi-bin/korp.cgi?cqp=%5Blemma+%3D+\""+query+"\"%5D&sort=keyword&command=query&indent=2&defaultcontext=0+words&show=lemma&start=0&end="+end+"&show_struct=text_dateto,text_datefrom&corpus="+corpus)
json_data = response.json()
print("Hit count "+str(json_data['hits'])+" should be below our end ("+str(end)+") if we are to get all data.")
print("Actually got "+str(len(json_data['kwic']))+" hits.")
json_data
```
```python
lemmaYearCountDict = dict()
for match in json_data['kwic']:
    lemma = match['tokens'][0]['lemma']
    year = match['structs']['text_datefrom'][:6]
    if not lemma in lemmaYearCountDict:
        lemmaYearCountDict[lemma]=dict()
    if not year in lemmaYearCountDict[lemma]:
        lemmaYearCountDict[lemma][year]=1
    else:
        lemmaYearCountDict[lemma][year]+=1
lemmaYearCountDict
```
```python
import pandas as pd
import numpy as np

lemmaFrame = pd.DataFrame(lemmaYearCountDict)

lemmaFrame = lemmaFrame.replace(np.nan,0).T
print('Frame dimensions: '+str(lemmaFrame.columns.size)+"x"+str(lemmaFrame.index.size))
lemmaFrame.head()
```
```python
import matplotlib.pyplot as plt
%matplotlib inline
from pylab import rcParams
rcParams['figure.figsize'] = 15, 10
import matplotlib.pyplot as plt
plt.locator_params(axis = 'x', nbins = 20)
lemmaFrame.sum().plot()
```
which gives me the following graph in which I can locate peaks in mentions responding quite nicely to new turns in the Guggenheim project (a history of which can be seen on [YLE news](http://yle.fi/uutiset/18-50706 "Title") or just [Wikipedia](https://en.wikipedia.org/wiki/Guggenheim_Helsinki_Plan "Title")), biggest peak corresponding to the first announcement and following ones to new turns in the project, mostly when it's future was being discussed in e.g. the city government.
![Guggenheim](https://s27.postimg.org/z4uzxy2cz/download_1.png "Guggemheim")
I made a similar graph for visual arts by quering for lemma "kuvataide" (fine arts) to compare if these peaks could be seen in the more general art related discussions.  
![Kuvataide](https://s30.postimg.org/enlj33zvl/download.png "Kuvataide")
To back these up, I wanted to make a complementary wordcloud for getting a better idea of the style of the conversations. I used Python again to get the search results with the following code

```python
import requests

response_p = requests.get("https://korp.csc.fi/cgi-bin/korp.cgi?command=query&cqp=%5Blemma+%3D+%22gugg.*%22%5D&corpus=s24_001,s24_002,s24_003,s24_004,s24_005,s24_006,s24_007,s24_008,s24_009,s24_010&start=0&end=15000")
json_data_p = response_p.json()
kwic_p = json_data_p["kwic"]

saneet_p = []

# imports only the tokens
for tietue in kwic_p:
    for alkio in tietue["tokens"]:
        for sane in alkio:
            saneet_p.append(alkio[sane])

# saves the tokens in a file
with open("saneet_gugge.txt", "w") as out_file_p:
    for s in range(len(saneet_p)):
        out_string = ""
        out_string += str(saneet_p[s]) + " "
        out_file_p.write(out_string)

out_file_p.close()
```  
I then used R to first preprocess the file and then tried to create a wordcloud of it with the following code referenced from [here](https://rstudio-pubs-static.s3.amazonaws.com/31867_8236987cf0a8444e962ccd2aec46d9c3.html "here").

```R
cname <- file.path("~", "Desktop", "texts")   
cname   
dir(cname)
library(tm)
encoding = "UTF-8"
  
docs <- Corpus(DirSource(cname))  
docs <- tm_map(docs, removePunctuation)
docs <- tm_map(docs, removeWords, stopwords("finnish"))
docs <- tm_map(docs, removeNumbers)
docs <- tm_map(docs, tolower)
for(w in seq(docs))
{
    docs[[w]] <- gsub("g\\S*", "", docs[[w]])
    docs[[w]] <- gsub("myös\\S*", "", docs[[w]])
}

docs <- tm_map(docs, stripWhitespace)
docs <- tm_map(docs, PlainTextDocument)

dtm <- DocumentTermMatrix(docs)

tdm <- TermDocumentMatrix(docs)
freq <- colSums(as.matrix(dtm))
ord <- order(freq)

dtms <- removeSparseTerms(dtm, 0.1)

freq <- colSums(as.matrix(dtms))
freq <- sort(colSums(as.matrix(dtm)), decreasing=TRUE)
library(ggplot2)

wf <- data.frame(word=names(freq), freq=freq)

p <- ggplot(subset(wf, freq>15), aes(word, freq))    
p <- p + geom_bar(stat="identity")   
p <- p + theme(axis.text.x=element_text(angle=45, hjust=1))
library(wordcloud)

set.seed(142)   
wordcloud(names(freq), freq, min.freq=15)
```
For one reason or another (might have been the version of R - or not, I eventually ran out of time to find out), I couldn't get nordic characters to work in this, so basically none of the finnish stopwords were removed either, but this is what I got
![wordcloud](https://s30.postimg.org/kl2ge7v81/Rplot.png "guggis")

### What the data shows

Just looking at the first graphs, it is easy to see the peaks in mentions every time the Guggenheim project, or its different steps, have been featured in the news. The project was first announced in early 2011 slowly kicking off the first and by far most obvious peak ending with the rejection of the original plan. The second bigger peak comes in about mid-2013 with the second museum proposal, it then seems to peek again with the beginning of the architectural competion related to the museum plan, and again when the competion winner was announced. The graph also shows a growing trend at its very end when the project was rejected for the second time in September 2016. Unfortunately, its re-emergence again in November 2016 and final (?) rejection are left out from the data.

Comparing this to the graph formed on *kuvataide*, it can be easily noted that the peaks that can be perceived in the Guggenheim graph are not just not repeating in this data, but at times almost reversed. This would seem to suggest that the two subjects are taken or handled separately in the Suomi24 discussion forum, and that the Guggenheim project is probably seen, as it has also been mostly presented in media, as an economic and perhaps a political issue and also discussed in those terms.

Looking at the produced wordcloud on the other hand some words with meaning seem to pop out such as  

* (jussi) pajunen
* rahat & rahaa
* veronmaksajien
* valtion
* kansa  

which all would seem to suggest same kind of conclusions as the first two graphs - the discussion being done around economics and politics. I also made a similar worldcloud for *kuvataide* to see a more general tone in the discussions concerning visual arts, mostly getting back results relating to music, literature and even sports.

Now these would seem to suggest the pessimistic conclusion that fine arts have failed to have an effect on discussions of economic, political and societal matters at this level. Then again, Suomi24 is a pretty specific kind of a forum, giving some opinions an unproportionally big voice. Other than that, the results from searching for *kuvataide* really show discussions of art related hobbies rather than fine arts. Also, it is no wonder that the matter of Guggenheim should be taken mostly in terms of economics, since that is the way it has been presented in media aswell. 