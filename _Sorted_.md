# Home

## Admin
```dataview
LIST FROM #admin SORT file.cdate
```
## Children
```dataview
LIST FROM #children SORT file.cdate
```
## Tech Tips

```dataview
LIST FROM #tech-tip 
SORT file.cdate
```
## Recipes
```dataview
LIST L.text 
FROM #recipe 
FLATTEN file.lists AS L 
WHERE contains(L.tags, "#recipe")
```
## Games
```dataview
LIST FROM #ps5 SORT file.cdate
```
# Work

## Contiamo
```dataview
LIST FROM #contiamo 
SORT file.ctime ASC
```
## CBRE
```dataview
LIST FROM #cbre 
SORT file.ctime ASC
```
## Telekom
```dataview
LIST FROM #telekom 
SORT file.ctime ASC
```
## Gautzsch
```dataview
LIST FROM #gautzsch SORT file.cdate
```