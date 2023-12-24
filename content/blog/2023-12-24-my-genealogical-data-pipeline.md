---
title: My Genealogical Data Pipeline
date: 2023-12-24T11:07:10.781Z
description: How I migrated data from a Aunt Ruth's notes to a neo4j database.
draft: false
showHero: true
showComments: true
thumbnail: /img/tree.jpeg
tags:
  - genealogy
  - data
  - gedcom
  - neo4j
  - graph database
---
A while back I started doing some genealogical research as a kind of pastime. At some point during this journey, I discovered that my Great-Aunt Ruth's had kept a database of genealogical data in some old, proprietary software. I didn't have any desire to buy said software, so I asked if there was anyway that the data could be exported. My cousin (once removed), Dave, replied that it could exported as a gedcom file. He sent me such a file.

A [gedcom](https://gedcom.io/about/) file (for a full explanation see link), is a format invented by the Mormon church to store genealogical data. It's plain text document that looks kind of like COBOL. It's rather inscrutable. 

Luckily there is a python library named [python-gedcom](https://gedcom.nickreynke.dev/gedcom/index.html) that can parse just such a file. Whatever I'm going to do next while dealing with the gedcom file is based on the documentation of the python-gedcom library linked above.

What I wanted to in particular, was do extract the genealogical data, such as parents, siblings, children, birth years, and death years - but, more importantly, to get Aunt Ruth's notes on relevant people. Aunt Ruth was a very smart an accomplished lady, she was a practicing psychologist in New Mexico and was an expert researcher - so her notes were the real prize!

I did this in a a Google Colab notebook. So the first thing I did was put my gedcom file in Google Drive and mount that drive. This was done like this:

```python
from google.colab import drive

drive.mount('/content/drive')
```

You also need to install the `python-gedcom` library:

```ipynb
!pip install python-gedcom
```

Next I made an instance of a gedcom parser instance, parsed said gedcom file, and got a list of all the root child elements of the gedcom file. 

One thing I discovered is that a gedcom file has several types of elements. I wanted to first filter out the elements representing people, so I made another list, only containing individual people:

```python
from gedcom.parser import Parser
from gedcom.element.individual import IndividualElement

gedcom_parser = Parser()
# Example of data being parsed with 'False' to disable strict parsing.
# if you want strict parsing, just don't put in a second argument.
gedcom_parser.parse_file('content/drive/My Drive/path_to_my_file.ged', False)

root_child_elements = gedcom_parser.get_root_child_elements()
individual_elements = [element for element 
                       in root_child_elements 
                       if isinstance(element, IndividualElement)]
```

I also discovered that each gedcom element has it's own unique id, called a "pointer" (maybe has something to do with the implementation - like a pointer in memory?). These pointers are how other elements are referenced. As such, I needed an easy way to find an element given a pointer. As such, I made the following dictionary:

```python
indexed_elements = {e.get_pointer(): e for e in root_child_elements}
```

Then I made a function for getting the data I wanted from the gedcom file:

```python
def get_personal_data(person: IndividualElement) -> dict:
    first_name, last_name = person.get_name()
    father, mother = [None, None] # In case there are no parents
    parents = gedcom_parser.get_parents(person)
    notes = None  # In case there are no notes
    if len(parents) == 1:
        father = parents[0].get_pointer() # override None for father
    elif len(parents) == 2:
        father, mother = [p.get_pointer() for p in parents]
    # Check here if Aunt Ruth had notes on this person ;)
    for child_element in person.get_child_elements():
        if child_element.get_tag() == "NOTE":
            note_pointer = child_element.get_multi_line_value()
            notes = indexed_elements.get(note_pointer).get_multi_line_value()

    return {
        "_id": person.get_pointer(),
        "first_name": first_name,
        "last_name": last_name,
        "father": father,
        "mother": mother,
        "birth_year": person.get_birth_year(),
        "died": person.get_death_year(),
        "burial_data": person.get_burial_data(),
        "birth_data": person.get_birth_data(),
        "notes": notes
        }
```

Next, I went through all the individuals and put them in a pandas data frame like this:

```python
import pandas as pd

data = [get_personal_data(person) for person in individuals]
df = pd.DataFrame(data)

df.set_index("_id")
```

At this point, I already had a dataframe with a lot of info, and all of Aunt Ruth's notes on each person. \
\
Here's an example of an entry of a person with Aunt Ruth's notes in the dataframe:

| _id  | first_name      | last_name | ... | notes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ---- | --------------- | --------- | --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| @I1@ | Avraham Gershel | Slutsky   | ... | The American great-grandchildren of Avraham Slutsky were told that their parents came from Russia. On all of the documents (e.g. marriage certificates, naturalization papers, death certificates) that we have, Russia is listed as their country of origin. However, Eugene Sloutsky who lives in Moscow and Mikhail Belikov, son of Chaya Ita Slutsky Belikov, (he also who lives in Moscow), tells us that the family came from Parafievka in Chernigov Gubernia, Ukraine. Teresa told her daughter that they came from Chernigov Gubernia... |