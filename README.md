Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item: https://www.wikidata.org/wiki/Q373548
contains all the data expected already, although Jeanine Áñez was not
yet at preferred rank.

Step 2: Tracking page
=====================

PositionHolderHistory already exists; current version is
https://www.wikidata.org/w/index.php?title=Talk:Q852448&oldid=1238279644
with 21 dated memberships and 79 undated; and 104 warnings.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js)
to configure the Item ID and source URL.

Step 4: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Step 5: Scrape
==============

Comparison/source = https://en.wikipedia.org/wiki/President_of_Nicaragua

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

Small tweaks needed, but trivial to scrape.

I had to manually combine the three terms of Evo Morales, though, as
Wikidata already had those as a single statement.

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

11 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/d228ffe137064

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

52 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/6169791a8bd87/

Step 8: Refresh the Tracking Page
=================================

New version at https://www.wikidata.org/w/index.php?title=Talk:Q852448&oldid=1238512968

I was deliberately only taking the low hanging-fruit of
easily-accessible start+end dates here, so there's still quite a bit of
work to be done after this.
