# EsutoniaGoDesu database design
This is a short description of the EGD PostgreSQL database.
Further info about the function of each database object is in SQL comments.

## Schemas
There are altogether 10 schemas in the database.

- **ac**: Access control. Controling access to API resources, enforcing policies and auditing usage.

- **core**: Core10k, Core6k and ILO vocabulary lists. Core data is based on respective Anki decks. Audio is stored in DB as well.
ILO list contains all words from the yellow pocket dictionary. Each word has been normalized to account for its kanji composition.
The entire Core10k list uses 2191 kanjis, Core6k - 1446, ILO - 1555.
- **estwn**: TEKSaurus data schema reduced to just 3 classes: example->variant->word_meaning.
- **freq**: various frequency lists. Used in test generation module and for creating translation queue.
- **heisig**: Doth RTK4 and RTK6 lists complemented with top 2 Koohii stories and various other information.
- **jmdict**: JP-ET dictionary.
- **jmdict_en**: JP-EN dictionary. This is being used in the article- and test generation module as JP-ET dictionary is not comprehensive enough.
- **kanjidic2**: all about Kanjis. Stroke diagrams are in public.image
- **public**: EsutoniaGoDesu specific data.
- **test**: serves data for article- and test generation module.

## JMDict gochas

JMDict identifiers have a bad habit of changing from time-to-time, meaning that links between different versions using entr.id is not possible.
If both English and Estonian translations of the same entr are needed, X should be used.


## Class diagrams
### AC
<img src="https://raw.githubusercontent.com/esutoniagodesu/egd-db/master/class-diagrams/schema/ac.png"/>




