# SPark


**Qual o objetivo do comando cache em Spark?**

O uso do comando **cache** ajuda a melhorar a eficiência do código nesse tipo de cenário, pois permite que resultados intermediários de operações *lazy* 
possam ser armazenados e reutilizados repetidamente. Pois maior parte das operações em um RDD são *lazy*, o que significa que, na prática, resultam apenas em uma 
abstração para um conjunto de instruções a serem executadas. 
Essas operações são realmente executadas através de ações, que não são operações *lazy*, pois podem ser avaliadas a partir da obtenção de valores do RDD. 
Por exemplo tornar códigos iterativos mais ineficientes, pois ações executadas repetidamente sobre um mesmo conjunto de dados disparam a ação de todas as operações *lazy* 
necessárias em cada uma das iterações, mesmo que resultados intermediários sejam iguais, como no caso da leitura de um arquivo. 


**O mesmo código implementado em Spark é normalmente mais rápido que a implementação equivalente em MapReduce. Por quê?**

Existem vários fatores no desenho dessas ferramentas que tornam as aplicações desenvolvidas em MapReduce geralmente mais lentas que aquelas que utilizam  Spark. 
Um desses fatores é o uso de memória. E necessario rodar vários *jobs* MapReduce em sequência em vez de um único *job*. 
Ao usar MapReduce, o resultado de cada *job* é escrito em disco, e precisa ser lido novamente do disco quando passado ao *job* seguinte. Spark, por outro lado, 
permite que resultados intermediários sejam passados diretamente entre as operações a serem executadas através do *caching* desses dados em memória, 
ou até mesmo que diversas operações possam ser executadas sobre um mesmo conjunto de dados em *cache*, reduzindo a necessidade de escrita/leitura em disco. 
Adicionalmente, mesmo em cenários onde ocorre a execução de apenas um *job*, o uso de Spark tende a ter desempenho superior ao MapReduce. Isso ocorre porque *jobs* 
Spark podem ser iniciados mais rapidamente, pois para cada *job* MapReduce uma nova instância da JVM é iniciada, 
enquanto Spark mantém a JVM em constantemente em execução em cada nó, precisando apenas iniciar uma nova *thread*, que é um processo extremamente mais rápido.


**Qual é a função do SparkContext ?**

O SparkContext funciona como um cliente do ambiente de execução Spark. Através dele, 
passam-se as configurações que vão ser utilizadas na alocação de recursos, como memória e processadores, pelos *executors*. 
Também usa-se o SparkContext para criar RDDs, colocar *jobs* em execução, criar variáveis de *broadcast* e acumuladores.


**Explique com suas palavras o que é Resilient Distributed Datasets (RDD)**

RDDs são as principais abstrações de dados do Spark. Eles são chamados *Resilient* por serem tolerantes à falhas, 
isto é são capazes de recomputar partes de dados perdidas devido as falhas nos *Distributed* porque podem estar divididos em partições 
através de diferentes nós em um cluster. Além dessas características, podem ser destacadas : RDDs são imutáveis, são objetos para leitura apenas, 
e só podem ser mudados através de transformações que resultam na criação de novos RDDs; Eles podem ser operados em paralelo, isto é, 
operações podem ser executadas sobre diferentes partições de um mesmo RDD ao mesmo tempo; RDDs são avaliados de forma "preguiçosa", 
de forma que os dados só ficam acessíveis e só são transformados quando alguma ação é executada (como mencionado na primeira questão); 
além disso RDDs têm seus valores categorizados em tipos, como números inteiros ou de ponto flutuante, strings, pares...


**GroupByKey é menos eficiente que reduceByKey em grandes dataset. Por quê?**

Quando fazendo uma agregação utilizando reduceByKey, Spark sabe que pode realizar a operação passada como parâmetro em todos os elementos de mesma chave em cada 
partição para obter um resultado parcial antes de passar esses dados para os executores que vão calcular o resultado final, resultando em um conjunto menor de dados sendo 
transferido. Por outro lado, ao usar groupByKey e aplicar a agregação em seguida, o cálculo de resultados parciais não é realizado,
dessa forma um volume muito maior de dados é desnecessariamente transferido através dos executores podendo, inclusive, 
ser maior que a quantidade de memória disponível para o mesmo, o que cria a necessidade de escrita dos dados em disco e resulta em um impacto negativo bastante 
significante na performance.


** Explique o que o código Scala abaixo faz **
```
1. val textFile = sc . textFile ( "hdfs://..." )
2. val counts = textFile . flatMap ( line => line . split ( " " ))
3.           . map ( word => ( word , 1 ))
4.           . reduceByKey ( _ + _ )
5. counts . saveAsTextFile ( "hdfs://..." )
```

Nesse código, um arquivo-texto é lido (linha 1). Em seguida, cada linha é "quebrada" em uma sequência de palavras e as sequencias correspondentes a cada linha são 
transformadas em uma única coleção de palavras (2). Cada palavra é então transformada em um mapeamente de chave-valor, com chave igual à própria palavra e valor 1 (3). 
Esses valores são agregados por chave, através da operação de soma (4).
Por fim, o RDD com a contagem de cada palavra é salvo em um arquivo texto (5).
