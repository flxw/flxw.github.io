+++
date = 2021-02-28
title = "A beancount tutorial and framework"
description = "Getting beancount to run and analyze your transactions"
tags = ["beancount"]
+++

As a financially avid reader, I am sure that you can roughly say how much money you need for a month.
You can assist your gut feeling with tools like Kmymoney, Homebank, GnuCash et al. to know exactly how much it is that you spend and earn per month.
However, the aforementioned tools have a common shortcoming, at least in my opinion:
They use different data formats to store data.
If the software reaches end of life, you could lose months or even years of transactional data because you might not be able to extract it from the old storage format.

Enter [Plain Text Accounting](https://plaintextaccounting.org/) (PTA).
A text file will store your transactions using a well-defined format.
No matter what happens to the analytics software, you will always be able to manipulate and convert from text - solving the problem above.

PTA tools tend to be designed for the command line, which can be somewhat counter-intuitive if you expect graphs and visualizations of the GUI tools.
In this article I will show you how to easily work with [beancount](https://beancount.github.io/) and visualize your numbers nicely.


# Directory setup
First, you install [beancount](https://beancount.github.io/) from any source that works for your system.

Second, create a file and folder structure like so:
```bash
importers/
raw/
main.beancount
import.config
```

Third, open `main.beancount` and open some accounts. You can use [this article](https://docs.google.com/document/d/1Tss0IEzEyAPuKSGeNsfNgb0BfiW2ZHyP5nCFBW1uWlk) as a guide. It doesn't have to be perfect from the start, you can always retry.

Fourth, create folders for your bank accounts inside `raw` and create empty beanfiles for them.

Fifth, use the `include` command inside `main.beancount` to pull in the account-specific beanfiles your just created.

If you're like me, you have two accounts where money shuttle back and forth. Then you would have a folder structure similar to the following:
```bash
import.config
importers/
main.beancount
n26.beancount
sparkasse.beancount
raw
 ├── N26
 └── Sparkasse               
```

# Importer setup
As mentioned in the introduction, I perceive the big strength of PTA that you can process anything.
For this job, beancount offers a so-called [Importer API](https://beancount.github.io/docs/importing_external_data.html#example-importers).
You can either write your own Python importer or copy and expand on one from Github.
I used the following two in my projects:
* [https://gitlab.hasi.it/seth/beancount-csv-camt]
* [https://github.com/siddhantgoel/beancount-n26]

Either install the importer via `pip` or make it importable for Python from the `importers/` directory.
It will process your input files line-by-line and create the appropriate accountin entries.

After having found and configured an importer, you can wire it up quite easily in `import.config`. Here is how it looks for me:

```python
import sys
from os import path
sys.path.insert(0, path.join(path.dirname(__file__))) 

from importers.n26 import N26Importer
from importers.csv_camt import CsvCamtImporter

CONFIG = [
    N26Importer(
        'XXXXXXMYIBANXXXXXXXXX',
        'Assets:N26',
        language='de',
        file_encoding='utf-8',
    ),
    CsvCamtImporter(
        'XXXXXXMYIBAN2XXXXXXXX',
        'Assets:Sparkasse'
    )
]
```

This tells beancount the following:
"Please use these two importers when importing transactions.
The importers will tell you whether they are capable of importing a given file, so the right one will get selected automatically."

# Data ingestion
A month has passed while you tried to set this up, so now you have a month's worth of data ;-)

Download it as CSV, and save it under `raw/n26-01-21.csv`.
Then, you can append the plain text ledger to the `n26.beancount` file:
```bash
bean-extract import.config raw/n26-01-21.csv >> n26.beancount
```

I recommend checking the output in the beginning, as categories could be wrong. Maybe you notice some patterns that can help you automatically categorize transactions inside the importer. I did that and it saves quite a bit of work.

# Data visualization
Here is why I picked beancount: [fava](https://beancount.github.io/fava/).
It's a great visualization frontend for beancount ledgers. After adding a month's worth of data, I just fire it up and can analyze what happened.

If you'd like to do so, too, simply install it and run:
```bash
fava main.beancount
```
