# Personal
```dataview
LIST from "Personal"
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