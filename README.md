![Family tree](https://github.com/Jakub-Glab/img/blob/main/image.png?raw=true)

# 1. Dziadków płci męskiej za pomocą relacji **hasGrandparent**

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT DISTINCT ?grandfather
WHERE
{
?subject ontology:hasGrandparent ?grandfather .
?grandfather a ontology:Man .
}

```

# 2. Wszystkie kuzynki osoby _M20_

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT DISTINCT ?cousin
WHERE
{
  ?cousin a ontology:Woman .
  ?cousin ontology:hasCousin <http://a.com/ontology#M20> .
}

```

# 3. Wszystkich męskich kuzynów osoby _M20_

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT DISTINCT ?cousin
WHERE
{
  ?cousin a ontology:Man .
  ?cousin ontology:hasCousin <http://a.com/ontology#M20> .
}

```

# 4. Małżeństwa, które spłodziły 2 córki

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT DISTINCT ?parent1 ?parent2
WHERE
{
  ?parent1 a ontology:Woman .
  ?parent1 ontology:hasConsort ?parent2 .
  ?parent1 ontology:hasDaughter ?daughter1 .
  ?parent2 ontology:hasDaughter ?daughter2 .
  FILTER (?daughter1 != ?daughter2)
}

```

# 5. Małżeństwa, które spłodziły zarówno kobietę jak i mężczyznę

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT DISTINCT ?parent1 ?parent2
WHERE
{
  ?parent1 ontology:hasConsort ?parent2 .
  ?parent1 ontology:hasChild ?child1 .
  ?parent2 ontology:hasChild ?child2 .
  ?child1 a ontology:Man .
  ?child2 a ontology:Woman .
}

```

# 6. Łańcuch: wnuk–dziadek–prababcia

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT ?grandchild ?grandfather ?greatgrandmother
WHERE
{
  ?grandchild ontology:hasGrandfather ?grandfather .
  ?greatgrandmother ontology:hasChild ?grandfather .
  ?greatgrandmother a ontology:Woman .
}

```

# 7. Wszystkich możliwych informacji o obiekcie _M20_

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT ?property ?value
WHERE
{
  <http://a.com/ontology#M20> ?property ?value .
}

```

# 8. Wszystkich relacji, które łączą się z tzw. _blank node_

```SQL

SELECT ?subject ?predicate ?blankNode
WHERE
{
  ?subject ?predicate ?blankNode .
  FILTER (isBlank(?blankNode))
}

```

# 9. Wszystkie kobiety, które mają syna, a nie mają córki

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT DISTINCT ?woman
WHERE
{
  ?woman a ontology:Woman .
  ?woman ontology:hasChild ?son .
  ?son a ontology:Man .
  OPTIONAL { ?woman ontology:hasChild ?daughter . ?daughter a ontology:Woman . }
FILTER (!bound(?daughter))
}

```

# 10. Linie męskich potomków (mężczyzna-mężczyzna-...-mężczyzna)

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT ?man1 ?man2 ?man3
WHERE
{
  ?man1 a ontology:Man .
  ?man1 ontology:hasChild ?man2 .
  ?man2 a ontology:Man .
  OPTIONAL
  {
    ?man2 ontology:hasChild ?man3 .
    ?man3 a ontology:Man .
  }
}

```

# 11. Osób będących "liśćmi" w drzewie (ostatni poziom: _F30, F31,_ ...)

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT DISTINCT ?leaf
WHERE
{
?leaf a ontology:Person .
FILTER NOT EXISTS { ?leaf ontology:hasChild ?child }
FILTER EXISTS { ?leaf ontology:hasGrandparent ?grandparent }
}

```

# 12. Osób znajdujących się na poziomie 1 (_M13, F10, F11,_ ...)

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT DISTINCT ?person
WHERE
{
  ?person a ontology:Person .

  FILTER NOT EXISTS { ?person ontology:hasGrandparent ?grandparent }
  FILTER ( EXISTS { ?person ontology:hasParent ?parent } ||
           EXISTS { ?person ontology:hasConsort ?consort .
                    ?consort a ontology:Person .
                    FILTER NOT EXISTS { ?consort ontology:hasGrandparent ?grandparent }
                    FILTER EXISTS { ?consort ontology:hasParent ?parent }
                  }
         )
}

```

# 13. Osób znajdujących się na poziomie 1, ale rezultaty zostają ograniczone do drugiej czwórki wyników (_F12, M11, M12_ i _F13_)

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT DISTINCT ?person
WHERE {
  ?person a ontology:Person .
  { ?person ontology:hasParent ontology:M01 }
  UNION
  {
    ?person ontology:hasConsort ?consort .
    ?consort a ontology:Man .
    ?consort ontology:hasParent ontology:M01 .
  }
  FILTER NOT EXISTS {
    ?person ontology:hasParent ?parent .
    ?parent ontology:hasSex ontology:Male .
    ?parent ontology:hasParent ontology:M01 .
    FILTER (?parent != ontology:M01)
  }
}

```

# 14. Wszystkich kobiet poprzez analizę nazwy obiektu (np. _F10_, gdzie F to Female)

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT ?woman
WHERE
{
?woman a ontology:Woman .
FILTER (regex(str(?woman), "^http://a.com/ontology#F"))
}

```

# 15. Zapytanie dotyczące wszystkich osób, które nie są w związku (nie mają wspólnych dzieci) za pomocą **NOT EXISTS** oraz **MINUS**

```SQL

PREFIX ontology: <http://a.com/ontology#>
SELECT DISTINCT ?person
WHERE {
  ?person a ontology:Person .
  FILTER NOT EXISTS {
    ?person ontology:hasConsort ?consort .
    ?consort ontology:hasChild ?child .
    ?child a ontology:Person .
    ?child2 a ontology:Person .
    FILTER (?child != ?child2)
  }

  MINUS {
    ?person ontology:hasChild ?child .
    ?child a ontology:Person .
    ?child2 a ontology:Person .
    FILTER (?child != ?child2)
  }
}

```
