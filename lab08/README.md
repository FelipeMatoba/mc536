## Tarefas com Publicações

## Questão 1
Construa uma comando SELECT que retorne dados equivalentes a este XPath
~~~xquery
//individuo[idade>20]/@nome
~~~

### Resolução
~~~xquery
SELECT individuo.nome 
FROM fichariodoc
WHERE individuo.idade > 20
~~~

## Questão 2
Qual a outra maneira de escrever esta query sem o where?

~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~
### Resolução
~~~sql
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')

for $i in ($fichariodoc//individuo[idade>17])
return $i
~~~

## Questão 3
Escreva uma consulta SQL equivalente ao XQuery:
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')

for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~

### Resolução
~~~sql
SELECT individuo.nome 
FROM fichariodoc
WHERE individuo.idade > 17
~~~

## Questão 4
Retorne quantas publicações são posteriores ao ano de 2011.

### Resolução
~~~xquery
let $pub := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')
return count($pub//publication[year>2011])  
~~~

## Questão 5
Retorne a categoria cujo `<label>` em inglês seja 'e-Science Domain'.

### Resolução
~~~xquery
let $pub := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')
let $cat := ($pub//categories[@catkey="subject"]//category)
for $c in $cat
where $c/label[@lang = "en-US"] = "e-Science Domain"
return $c
~~~

## Questão 6
Retorne as publicações associadas à categoria cujo `<label>` em inglês seja 'e-Science Domain'. A associação entre o label e a key da categoria deve ser feita na consulta.

### Resolução
~~~xquery
let $pub := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')
let $cat := ($pub//categories[@catkey="subject"]//category)
for $c in $cat
where $c/label[@lang = "en-US"] = "e-Science Domain"
let $publications := ($pub//publication)
for $p in $publications
where $p/key/text() = $c/@key
return $p
~~~

## Tarefas com DRON e PubChem

## Questão 1

Liste o nome de todas as classificações que estão apenas dois níveis imediatamente abaixo da raiz.

### Resolução
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
return data($dron/drug/drug//drug/@name)
~~~

## Questão 2

Apresente todas as classificações de um componente a sua escolha (diferente de `Acetylsalicylic Acid`) que estejam hierarquicamente dois níveis acima. Note que no exemplo dado em sala foi considerado um nível hierárquico acima.

### Resolução
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
for $d in ($dron//drug[drug/drug/@name = 'POTASSIUM CHLORATE'])
let $gr := $d/@name
group by $gr
return {data($gr), '&#xa;'}
~~~

## Questão 3

### Questão 3.1

Liste todos os códigos ChEBI dos componentes disponíveis.

#### Resolução
~~~xquery
let $data := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')
for $info in ($data/PC-DataSet//InformationList/Information//Synonym)
where substring($info/text(), 1, 5) = 'CHEBI'
let $gr := $info
group by $gr
order by $gr
return {data($gr), '&#xa;'}
~~~

### Questão 3.2

Liste todos os códigos ChEBI e primeiro nome (sinônimo) de cada um dos componentes disponíveis.

#### Resolução
~~~xquery
let $data := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')
for $info in ($data/PC-DataSet//InformationList/Information)
for $chebi in ($info//Synonym)
where substring($chebi/text(), 1, 5) = 'CHEBI'
let $gr := $chebi
group by $gr
order by $gr
return {data($info/Synonym[1]) ,data($gr), '&#xa;'}
~~~

### Questão 3.3

Para cada código ChEBI, liste os sinônimos e o nome que aparece para o mesmo componente no DRON (se existir). Para isso faça um JOIN com o DRON.

#### Resolução
~~~xquery
let $data := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')
for $info in ($data/PC-DataSet//InformationList/Information)
for $chebi in ($info//Synonym)
where substring($chebi/text(), 1, 5) = 'CHEBI'

let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
for $d in ($dron//drug)
where substring(data($d/@id), 38) = substring($chebi/text(), 7)

let $gr2 := data($d/@name)
let $gr := $chebi
group by $gr, $gr2
order by $gr

return {$gr2, '&#xa;', (data($info/Synonym), '&#xa;'),'&#xa;'}
~~~
