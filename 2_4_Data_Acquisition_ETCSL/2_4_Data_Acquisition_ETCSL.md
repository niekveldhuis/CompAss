## 2.4 Data Acquisition: ETCSL
Back to the main [COMPASS][] page.

Back to COMPASS Chapter 2

[TOC]
# NOTE: under construction

The Electronic Text Corpus of Sumerian Literature ([ETCSL][]) provides editions and translations of almost 400 Sumerian literary texts, mostly from the Old Babylonian period (around 1800 BCE). The project was led by Jeremy Black (Oxford University) and was active until 2006, when it was archived. Information about the project, its stages, products and collaborators may be found in the project's [About](http://etcsl.orinst.ox.ac.uk/edition2/general.php) page. By the time of its inception [ETCSL][] was a pioneering effort - the first large digital project in Assyriology, using well-structured data according to the standards and best practices of the time. [ETCSL][] allows for various kinds of searches in Sumerian and in English translation and provides lemmatization for each individual word. Numerous scholars contributed data sets to the [ETCSL][] project (see [Acknowledgements](http://etcsl.orinst.ox.ac.uk/edition2/credits.php#ack)). The availability of [ETCSL][] has fundamentally altered the study of Sumerian literature and has made this literature available for undergraduate teaching.

The original [ETCSL][] files in TEI XML are stored in the [Oxford Text Archive][] from where they can be downloaded as a ZIP file under a Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Unported License ([by-nc-sa 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/)). The copyright holders are Jeremy Black, Graham Cunningham, Jarle Ebeling, Esther Flückiger-Hawker, Eleanor Robson, Jon Taylor, and Gábor Zólyomi.

The [Oxford Text Archive][] page offers the following description:

> The Electronic Text Corpus of Sumerian Literature (ETCSL) comprises transliterations and English translations of 394 compositions attested on sources dating to the period from approximately 2100 to 1700 BCE. The compositions are divided into seven categories: ancient literary catalogues; narrative compositions; royal praise poetry and hymns to deities on behalf of rulers; literary letters and letter-prayers; divine and temple hymns; proverbs and proverb collections; and a more general category including compositions such as debates, dialogues and riddles. The numbering of the compositions within the corpus follows Miguel Civil's unpublished catalogue of Sumerian literature (etcslfullcat.html).Files with an initial c are composite transliterations (a reconstructed text editorially assembled from the extant exemplars but including substantive variants) in which the cuneiform signs are represented in the Roman alphabet. Files with an initial t are translations. The composite files include full references for the cuneiform sources and author-date references for the secondary sources (detailed in bibliography.xml). The composite and translation files are in XML and have been annotated according to the TEI guidelines. In terms of linguistic information, each word form in the composite transliterations has been assigned to a lexeme which is specified by a citation form, word class information and basic English translation.

Since [ETCSL][] is an archival site, the editions are not updated to reflect new text finds or new insights in the Sumerian language. Many of the [ETCSL][] editions were based on standard print editions that itself may have been 10 or 20 years old by the time they were digitized. Any computational analysis of the [ETCSL][] corpus will have to deal with the fact that: 

- the text may not represent the latest standard
- the [ETCSL][] corpus is extensive - but does not cover all of Sumerian literature known today

In terms of data acquisition, the way to deal with these limitations is to make the [ETCSL][] data as much as possible compatible with the data standards of the Open Richly Annotated Cuneiform Corpus ([ORACC][]). [ORACC][] is an active project where new or updated editions can be produced. If compatible, if [ETCSL][] and [ORACC][] data may be freely mixed and matched, then the [ETCSL][] data set can effectively be updated and expanded.

The [ETCSL][] text corpus was one of the core data sets for the development of of [ePSD1](http://psd.museum.upenn.edu/epsd1/index.html) and [ePSD2][] (currently in a Beta version) and this version of the [ETCSL][] data is available at [ePSD2/ETCSL][] and can be parsed with the ORACC parser, discussed in section 2.3. In order to include the data in ePSD the lemmatization is adapted there to [ORACC][] standards and thus this version of the [ETCSL][] dataset is fully compatible with any [ORACC][] dataset.

Parsing the original [ETCSL][] XML TEI files has, therefore, become somewhat redundant. The reason to include and discuss the [ETCSL][] parser here is, first, to offer users the opportunity to work with the original data set. The various transformations included in the current parser may be adapted and adjusted to reflect the preferences and research questions of the user. Second, [ETCSL][] distinguishes between main text, secondary text, and additional text, to reflect different types of variants between manuscripts (see below 2.4.4). The [ePSD2/ETCSL][] data set does not include this distinction. The output of the parser will indicate for each word whether it is "secondary" or "additional" (according to [ETCSL][] criteria) and offer the possibility to include such words or exclude them from the  analysis.

In order to achieve compatability between [ETCSL][] and [ORACC][] the code uses a number of equivalence dictionaries, that enable replacement of characters, words, or names. These equivalence dictionaries are stored in JSON format (for JSON see section 2.3) in the file `equivalancies.json`  in the directory `equivalencies`.

### 2.4.1 TEI XML format

The [ETCSL][] files as distributed by the [Oxford Text Archive][] are encoded in a dialect of XML (Extensible Markup Language) that is referred to as TEI (Text Encoding Initiative). In this encoding each word (in transliteration) is an *element* that is surrounded by `<w>` and `</w>` tags. Inside the start-tag the word may receive several attributes, encoded as name/value pairs:

```xml
<w form="mah-bi" lemma="mah" pos="V" label="to be majestic">mah-bi</w>
```

In parsing the [ETCSL][] files we will be looking for these `<w>` and `</w>` tags to isolate words and their attributes. Higher level tags identify lines (`<l>` and `</l>`) , versions, secondary text (found only in a minority of sources), etcetera.

The [ETCSL][] file set includes the [etcslmanual.html](http://etcsl.orinst.ox.ac.uk/edition2/etcslmanual.php) with explanations of the tags, their attributes, and their proper usage.

### 2.4.2 `lxml` and `Xpath`

There are several Python libraries specifically for parsing `XML`, among them the popular `ElementTree` and its twin `cElementTree`. The library `lxml` is largely compatible with `ElemnetTree` and `cElementTree` but differs from those in its full support of `Xpath`. `Xpath` is a language for finding and retrieving elements and attributes in XML trees. `Xpath` is not a program or a library, but a set of specifications that is implemented in a variety of software packages in different programming languages. 

In `Xpath` `'//w'`means: all the `w` nodes, wherever in the hierarchy of the `XML` tree. The expression may be used to create a list of all the `w` nodes with all of their associated attributes. The attributes of a node are addressed  with the `@` sign, so that `//w[@label]` refers to the `label` attributes of all the `w` nodes at any level in the hierarchy. 

```python
words = tree.xpath('//w')
labels = tree.xpath('//w[@label]')
```

`Xpath` also defines hundreds of functions. An important function is `'string()'` which will return the string value of a node or an attribute.  The `string()` function is usually applied to a single node. Once all `w` nodes are listed in the list `words` (with the code above) one may extract the transliteration and Guide Word (`label` in [ETCSL][]) of each word as follows:

```python
form_l = []
gw_l = []
for node in words:
    form = node.xpath('string(.)') 
    form_l.append(form)
    gw = node.xpath('string(@label)')
    gw_l.append(gw)
```

The dot in `node.xpath('string(.)')` refers to the current node.

For proper introductions to `Xpath` and `lxml` see the [Wikipedia](https://en.wikipedia.org/wiki/XPath) article on `Xpath` and the homepage of the [`lxml`](https://lxml.de/) library.

### 2.4.3 Pre-processing: HTML entities

Before the XML files can be parsed, it is necessary to remove character sequences that are not allowed in XML proper (so-called HTML entities). 

In non-transliteration contexts (bibliographies, text titles, etc.) [ETCSL][] uses so-called HTML entities to represent non-ASCII characters such as,  á, ü, or š. These entities are encoded with an opening ampersand (`&`) and a closing semicolon (`;`). For instance, `&C;` represents the character `Š`. The HTML entities are for the most part project-specific and are declared in the file `etcsl-sux.ent` which is part of the file package and is used by the [ETCSL][] project in the process of validating and parsing the XML for online publication.

For purposes of data acquisition these entities need to be resolved, because XML parsers will not recognize these sequences as valid XML. 

The key `ampersands`in the file `equivalences.json` includes a dictionary, listing all the HTML entities that appear in the [ETCSL][] files with their Unicode counterparts:

```json
{'&C;': 'Š',
 '&Ccedil;': 'Ç',
 '&Eacute;': 'É',
 '&G;': 'Ŋ',
 '&H;': 'H',
 '&Imacr;': 'Î',
 '&X;': 'X',
 '&aacute;': 'á',
 etc.  
```

This dictionary is used to replace each HTML entity with its unicode (UTF-8) counterpart in each of the data files (the original files are, of course, left untouched). The function `ampersands()` is called in the main process. 

```python
import json
with open("equivalencies/equivalencies.json") as f:
    eq = json.load(f)
def ampersands(x):
    for amp in eq["ampersands"]:
        x = x.replace(amp, eq["ampersands"][amp])
    return x
```



### 2.4.4 Pre-Processing: Additional Text and Secondary Text

In order to be able to preserve the [ETCSL][] distinctions between main text (the default), secondary text, and additional text, such information needs to be added as an attribute to each `w` node (word node). This must take place in pre-processing, before the `XML` files are parsed.

[ETCSL][] transliterations represent composite texts, put together (in most cases) from multiple exemplars. The editions include substantive variants, which are marked either as "additional" or as "secondary". Additional text consists of words or lines that are *added* to the text in a minority of sources. In the opening passage of [Inana's Descent to the Netherworld][], for instance, there is a list of temples that Inana leaves behind. One exemplar extends this list with eight more temples; in the composite text these lines are marked as "additional" and numbered lines 13A-13H. Secondary text, on the other hand, is variant text (words or lines) that are found in a minority of sources *instead of* the primary text. An example in [Inana's Descent to the Netherworld][] is lines 30-31, which are replaced by 30A-31A in one manuscript (text and translation as in [ETCSL][]):

| line | text                                       | translation                                                  |
| ---- | ------------------------------------------ | ------------------------------------------------------------ |
| 30   | sukkal e-ne-eĝ₃ sag₉-sag₉-ga-ĝu₁₀          | my minister who speaks fair words,                           |
| 31   | ra-gaba e-ne-eĝ₃ ge-en-gen₆-na-ĝu₁₀        | my escort who speaks trustworthy words                       |
| 30A  | [na] ga-e-de₅ na de₅-ĝu₁₀ /ḫe₂\\-[dab₅]    | I am going to give you instructions: my instructions must be followed; |
| 31A  | [inim] ga-ra-ab-dug₄ ĝizzal [ḫe₂-em-ši-ak] | I am going to say something to you: it must be observed      |

"Secondary text" and "additional text" can also consist of a single word and there are even cases of "additional text" within "additional text" (an additional word within an additional line).

In [ETCSL][] TEI XML secondary/additional text is introduced by a tag of the type:

```xml
<addSpan to="c141.v11" type="secondary"/>
```

or

```xml
<addSpan to="c141.v11" type="additional"/>
```

The number c141 represents the text number in [ETCSL][] (in this case [Inana's Descent to the Netherworld][], text c.1.4.1). The return to the primary text is indicated by a tag of the type:

```xml
<anchor id="c141.v11"/>
```

Note that the `id` attribute in the `anchor` tag is identical to the `to` attribute in the `addSpan` tag.

We can collect all the `w` tags (words) between `addSpan` and its corresponding `anchor` tag wih the following `xpath` expression:

```python
secondary = tree.xpath('//w[preceding::addSpan[@type="secondary"]/@to = following::anchor/@id]')
```

In the expression `preceding` and `following` are so-called `axes` (plural of `axis`) which describe the relationship of an element to another element in the tree. The expression means: get all `w` tags that are preceded by an `addSpan` tag and followed by an `anchor` tag. The `addSpan` tag has to have an attribute `type` with value `secondary` , and the value of the `to` attribute of this `addSpan` tag is to be equal to the `id` attribute of the following `anchor` tag.

Once we have collected all the "secondary" `w` tags this way, we can add an attribute to each of these words in the following way:

```python
for word in secondary:
    word.attrib["status"] = "secondary"
```

In the process of parsing we can retrieve this new `status` attribute to mark all of these words as `secondary`.

Since we can do exactly the same for "additional text" we can slightly adapt the above expression for use in a function:

```python
def mark_extra(tree, which):
    extra = tree.xpath('//w[preceding::addSpan[@type="' + which + '"]/@to = following::anchor/@id]')
    
    for word in extra:
        word.attrib["status"] = which
	return tree
```

In the main process the function `mark_extra()`is called with the entire `XML` tree as its first argument, and  "additional" or "secondary" as its second argument.

### 2.4.6 Gaps

Gaps of one or more lines in the composite text, due to damage to the original cuneiform tablet, is encoded as follows:

```xml
<gap extent="8 lines missing"/>
```

In order to be able to process this information and keep it at the right place in the data we will parse the `gap` tags together with the `l` (line) tags and process the gap as a line.



### 2.4.7 Parsing the XML Tree

The Python library `lxml`, includes a unit `etree`, specialized in parsing XML trees. The code basically works from the highest level of the hierarchy  to the lowest, in the following way:

```
text							parsetext()
	version						getversion()
		section					getsection()
			line				getline()
				word			getword()
```

The main process calls the function `parsetext()` which calls the pre-processing functions discussed in section 2.4.3. It then calls `getversion()`, which calls `getsection()`, which calls `getline()`, which, finally, calls `getword()`. Along the way a dictionary, `meta_d` collects information about text IDs, text names, version names, and line numbers. The function `getline()` adds a line reference number (`line_ref`), an integer, in order to be able to put  lines and gaps in their correct sequence (`line_no` is a string and will put line "11" before line "3"). The function `getword()` at the lowest level of this hierarchy, will create a dictionary for each word. This dictionary contains all the information about line number, section, version name, text name, etc. plus the lemmatization data of the single word.

Thus the word `šeŋ₆-ŋa₂` in the file `c.1.2.2.xml` ([Enlil and Sud](http://etcsl.orinst.ox.ac.uk/cgi-bin/etcsl.cgi?text=c.1.2.2&display=Crit&charenc=gcirc#)), in Version A, section A line 115, looks as follows in XML: 

```xml
<w form="cej6-ja2" lemma="cej6" pos="V" label="to be hot">cej6-ja2</w>
```

The dictionary that is created in `getword()` represents that same word as follows:

```python
{'id_text': 'c.1.2.2', 
 'text_name': 'Enlil and Sud',
 'version': 'A', 
 'lang': 'sux',
 'cf': 'šeŋ',
 'gw': 'cook',
 'pos': 'V/t',
 'form': 'šeŋ₆-ŋa₂',
 'line_no' : 'A115',
 'line_ref': 109,
 'extent': ''}
```

Note that in the process the transliteration and lemmatization data have been replaced by [epsd2][] style data. The function `tounicode()` replaces `j` by `ŋ` , `c` by `š`, etc. and substitutes Unicode subscript numbers for the regular numbers in sign indexes. The function `etcsl_to_oracc()` replaces [ETCSL][] style lemmatization with [epsd2][] style (`gw` 'cook' instead of `label= "to be hot"`). Both replacements use dictionaries in the `equivalencies.json` file.  

[ETCSL]:                               http://etcsl.orinst.ox.ac.uk
[ORACC]:                             http://oracc.org
[epsd2]:                               http://oracc.org/epsd2/sux
[epsd2/etcsl]: http://oracc.museum.upenn.edu/epsd2/etcsl/
[Inana's Descent to the Netherworld]: http://etcsl.orinst.ox.ac.uk/cgi-bin/etcsl.cgi?text=c.1.4.1&amp;amp;amp;display=Crit&amp;amp;amp;charenc=gcirc#
[Oxford Text Archive]:       http://ota.ox.ac.uk/desc/2518
[COMPASS]:	http://oracc.org/compass
