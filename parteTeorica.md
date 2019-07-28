# desafioSemantix
**Desafio Técnico** - Engenheiro de Dados 

**_Leonardo Damasio_**

## Parte Teórica

* Qual o objetivo do comando **cache** em Spark?

> O objetivo do comando `cache` é basicamente armazenar um resultado computado do RDD na memória, permitindo que futuramente tenhamos acesso sem que necessitemos reprocessá-lo. É um mecanismo muito útil quando queremos acessar determinados dados repetidamente, trazendo um aumento de performance significativo em códigos iterativos. Também é tolerante a falha, pois automaticamente recupera uma informação perdida de uma partição do RDD através da linhagem de instruções que previamente a originou. 

> Um comando que pode ser utilizado como similar ao cache é o `persist`. Enquanto cache utiliza por default o armazenamento apenas em memória (`MEMORY_ONLY`), o comando `persist` nos permite escolher se queremos armazenar de outros modos, como `MEMORY_ONLY`, que basicamente nos traria o mesmo resultado que o comando `cache`, ou de outros modos, como `MEMORY_ONLY_SER`, `MEMORY_AND_DISK`, `MEMORY_AND_DISK_SER`, `DISK_ONLY` ou `OFF_HEAP`, que ainda está em fase experimental para o Spark. 



* O mesmo código implementado em Spark é normalmente mais rápido que a implementação equivalente em MapReduce. Por quê?

> Isto acontece porque o MapReduce normalmente processa dados em disco (HDFS), enquanto o Spark processa tanto em disco quanto em memória. Ou seja, enquanto MapReduce realiza consultas frequentes ao HDFS, todo o processamento no Spark pode acontecer na memória. 

> Veja na simplificação abaixo:

![](https://www.xpand-it.com/wp-content/uploads/2019/06/meetup-spark-intro-data-sharing.png)

> Em compensação, gigantescos volumes de dados, nos quais não se é possível realizar todo o processo em memória, MapReduce ainda é utilizado. Geralmente, por consenso da comunidade, baseado em benchmarking de ambos, o Spark é utilizado para volumes de dados em até 1TB e MapReduce para volumos superiores a isto. Mas claro, tudo depende da capacidade do cluster à disposição. 

> Tudo isso faz com que MapReduce seja bem limitado quando trabalhamos com modelos iterativos, como, por exemplo, de Machine Learning. O que já não acontece no Spark, pois, através do uso de RDDs, armazenados em memória, o tempo de execução se torna bem menor.





* Qual é a função do **SparkContext** ?

> A função do `SparkContext` é servir como objeto de entrada às funcionalidades do Spark, servindo como cliente do ambiente de execução e permitindo a aplicação acessar o cluster, onde é possível informar algumas das configurações que utilizaremos, contendo, por exemplo, qual é o caminho para acessar o cluster (`Master`), qual o nome da aplicação (`appName`), como será feita a distribuição dos recursos como memória, executores, etc. (`Conf`), dentre outros. 

> É considerada por muitos como o coração do Spark, pois através desta função conseguimos criar RDDs, acessar os serviços do Spark e rodar jobs. 



* Explique com suas palavras o que é **Resilient Distributed Datasets** (RDD).

> É o objeto principal da programação em Spark. Trata-se de uma abstração de dados do Spark para sua manipulação. Ele representa um conjunto de dados, distribuídos ao longo dos nós de um cluster, permitindo sua operação de modo paralelo. Sobre estes datasets podemos realizar transformações, que nos permitem fazer operações como mapeamentos e filtros, e ações, como contagens e somatórias, retornando um valor. Apesar disto, os RDDs são imutáveis, então as transformações não agem no RDD em si, mas criam outro com a transformação efetuada.

> RDDs podem estar armazenados no HDFS (Hadoop Distributed File System) e em outros bancos de dados NoSQL, como HBase e Cassandra.

> São chamados assim pois realmente são resilientes, ou seja, tolerantes à falha. Permitindo que, caso uma partição desapareça ou seja danificada em algum nó do cluster, seja possível recuperar através da linhagem de modificações do RDD. 


* **GroupByKey** é menos eficiente que **reduceByKey** em grandes dataset. Por quê?

> Apesar de nos retornarem os mesmos resultados, o método de execução do `groupByKey` apenas combina os dados no output, enquanto o `reduceByKey` permite o Spark combinar os dados em cada executor antes de combiná-los, conseguindo ganhar eficiência no caso de uma grande quantidade de dados.

> Podemos notar a clara diferença nos exemplos abaixo:

![](https://techmagie.files.wordpress.com/2015/09/bp5.png?w=275&h=174&zoom=1.5)
![](https://techmagie.files.wordpress.com/2015/09/bp4.png?w=291&h=178&zoom=1.5)

> Isto pode não fazer muita diferença em pequenos datasets, mas quando falamos de Big Data, `reduceByKey` pode nos trazer um grande aumento de performance.

* Explique o que o código Scala abaixo faz.

```scala

val textFile = sc . textFile ( "hdfs://..." )
val counts = textFile . flatMap ( line => line . split ( " " ))
					. map ( word => ( word , 1 ))
					. reduceByKey ( _ + _ )
counts . saveAsTextFile ( "hdfs://..." )

```

> Dividindo em partes:


```scala
val textFile = sc . textFile ( "hdfs://..." )
```
> `val` para definir `textFile` como um valor fixo (objeto, assim como a documentação Scala nos diz "*Scala é uma linguagem puramente orientada a objetos no sentido que todo valor é um objeto.*" Fonte: [Scala Documentation](https://docs.scala-lang.org/pt-br/tutorials/tour/tour-of-scala.html.html) ).

> `sc` para chamar o `SparkContext` (`sc` aqui foi tratado como alias de `SparkContext`).

> Método `textFile` para ler o conteúdo do arquivo na URL, que está indicando um caminho do HDFS.

> *Resumindo*: Lê um arquivo do HDFS e armazena na RDD `textFile`.

```scala
val counts = textFile . flatMap ( line => line . split ( " " ))
					. map ( word => ( word , 1 ))
					. reduceByKey ( _ + _ )
```

> `val` para definir `counts` como um valor fixo (objeto).

> `textFile` agora é nossa RDD (objeto), na qual aplicamos uma transformação `.flatMap` que basicamente mapeia os dados com a função `split` por espaços (`" "`), devolvendo-os de forma o elemento linha em elementos palavras.

> `map` para atribuir a cada palavra desta lista um valor (`1`), transformando cada conjunto destes em uma tupla.

>`reduceByKey` para reduzir através do argumento `( _ + _ )`, o que permite eliminar as palavras duplicadas desta nova lista e contar o número de ocorrências no segundo elemento de cada tupla. Ou seja, realiza uma contagem de quantas vezes determinada palavra se repete.

> *Resumindo*: Cria uma uma nova RDD chamada `counts` contendo uma transformação na RDD `textFile`, trazendo suas informações agora em formato de lista, separando seus itens nos locais onde havia espaços e atribuindo-lhes um valor (`1`), criando tuplas e as reduzindo para realizar uma contagem do primeiro elemento da tupla, no segundo elemento.

```scala
counts . saveAsTextFile ( "hdfs://..." )
```

> Por final, utiliza a ação `saveAsTextFile` para salvar esta RDD (`counts`) em um arquivo de texto no local indicado no parâmetro URL, pertencente ao HDFS ( `( "hdfs://..." )` ).

>*Resumo do código*: **Contagem de palavras**



