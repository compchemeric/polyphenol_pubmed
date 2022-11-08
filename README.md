# polyphenol_pubmed
# Tiny script to query pubmed using the phenol explorer database

%pip install pymed
from pymed import PubMed
import pandas as pd
from IPython.display import Markdown, display
import regex as re
from google.colab import files

import numpy as np

def printmd(string):
    display(Markdown(string))

#!wget https://github.com/compchemeric/polyphenol_pubmed/blob/main/natural-compounds-classification.csv
%ls

mols = pd.read_csv('natural-compounds-classification.csv', sep=";") 
print(mols)   
mollist = mols['Compound Name'].tolist()

if True:
  number_of_pubs = []
  for mol in mollist:

    query_words = ['ampk', mol]

    pubmed = PubMed(tool="MyTool", email="my@email.address")

    #print(mol)

    try:
      results = pubmed.query(f'{query_words[0]} "{query_words[1]}"[Title/Abstract]', max_results=500)
      number_of_pubs.append(len(list(results)))
    except: 
      print('error with pubmed query')
      number_of_pubs.append(np.nan)

  np.save('n_pubs.npy', number_of_pubs)

n_pubs = np.load('n_pubs.npy')

d = {'Molecules': mollist,
     'Publications': n_pubs}

df = pd.DataFrame(d)
df.sort_values('Publications', ascending=False, inplace=True)
print(df.head(100))

if True:
  for i,article in enumerate(results):
    article_id = article.pubmed_id
    title = article.title
    for word in query_words:
      title = re.sub(f'{word}', f'**{word}**', title)

    publication_date = article.publication_date
    abstract = article.abstract
    authors = article.authors
    journal = article.journal

    printmd(f'{title} {publication_date} {journal}')
