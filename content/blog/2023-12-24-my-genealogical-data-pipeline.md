---
title: My Genealogical Data Pipeline
date: 2023-12-24T11:07:10.781Z
description: How I migrated data from a Aunt Ruth's notes to a neo4j database.
draft: true
showHero: true
showComments: true
thumbnail: /img/tree.jpeg
tags:
  - news
---
A while back I started doing some genealogical research as a kind of pastime. At some point during this journey, I discovered that my Great-Aunt Ruth's had kept a database of genealogical data in some old, proprietary software. I didn't have any desire to buy said software, so I asked if there was anyway that the data could be exported. My cousin (once removed), Dave, replied that it could exported as a gedcom file. He sent me such a file.

A [gedcom](https://gedcom.io/about/) file (for a full explanation see link), is a format invented by the Mormon church to store genealogical data. It's plain text document that looks kind of like COBOL. It's rather inscrutable. 

Luckily there is a python library named [python-gedcom](https://gedcom.nickreynke.dev/gedcom/index.html) that can parse just such a file. What I wanted to in particular, was do extract the genealogical data, such as parents, siblings, children, birth years, and death years - but, more importantly, to get Aunt Ruth's notes on relevant people. Aunt Ruth was a very smart an accomplished lady, she was a practicing psychologist in New Mexico and was an expert researcher - so her notes were the real prize!