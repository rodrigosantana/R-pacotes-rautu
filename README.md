# R-pacotes-rautu

Criação de pacotes para o R: uma rápida introdução



## Sumário

1. [Motivação][1]
2. [Estrutura de um pacote][2]
3. [`DESCRIPTION`: Caracterização do pacote][3]
4. [`R/`: Funções][4]
5. [`man/`: Documentação][5]
6. [`NAMESPACE`: Organização][6]
  * [Exportando funções][6.1]
  * [Importando funções][6.2]
7. [Workflow][7]
8. [Extra: integração contínua com Travis-CI][8]

## Motivação

Como já dizia [John M. Chambers][] quando criou a linguagem S
(predecessora do R), o objetivo da linguagem é

> "... to turn ideas into software, quickly and faithfully."

e transformar usuários em programadores de maneira rápida e
conveniente.

Se você já explorou um pouco mais do que o potencial básico do R,
certamente já criou sua própria função (ou várias funções) para fazer
alguma tarefa específica. A ideia é que se você tiver que realizar uma
tarefa muitas vezes, então deve criar uma função para fazer isso.

A criação de um pacote do R pode ter duas motivações principais:

* Agrupar e manter uma série de funções criadas por você e que são
  basicamente de uso próprio ou rotineiro.
* Compartilhar funções de novos métodos e/ou implementações que você
  tenha criado e queira disponibilizar para outras pessoas usarem.

Não raramente, um pacote de uso pessoal acaba se tornando útil para
outras pessoas e o autor disponibiliza para uso geral. O importante é
então saber como transformar suas funções em um pacote. E mais
importante ainda é saber como fazer isso se preocupando apenas com as
funções, ao invés de se preocupar com todo o processo de criação do
pacote.

A criação de um pacote no R pode ser feita de duas formas:

* Tradicional: usando os comandos padrões do R para criação de pacotes,
  como `R CMD build` para criar e `R CMD check` para conferir o pacote,
  ambos executados através de um terminal e não de uma sessão do R.
* Moderna: usando o pacote `devtools`, que possui funções para criar e
  conferir o pacote (inclusive para testar e documentar as funções) de
  dentro de uma sessão do R.

A maneira tradicional de criar pacotes no R é um pouco mais trabalhosa,
pois cada vez que você quiser checar se seu pacote está funcionando, é
necessário ir para um terminal e rodar os comandos por fora do R. Além
disso, se você quiser testar seu pacote após a instalação, será
necessário instalar de fato o pacote e carregá-lo com `library()`. O
grande problema é que durante o desenvolvimento de um pacote, esses
passos geralmente são repetidos muitas vezes, o que torna todo o
processo muito demorado.

O pacote `devtools` facilita todo esse processo, pois você pode
construir, checar, documentar (e conferir a documentação) de dentro da
própria sessão do R que você está desenvolvendo o pacote. Além disso
esse pacote também permite que, com uma função, você simule a instalação
do seu pacote na mesma sessão R, evitando ter que instalar e desinstalar
para testes todas as vezes.

Um outro aspecto fundamental de todo pacote é a sua documentação. Da
maneira tradicional, é necessário criar os arquivos `.Rd` e escrever
usando uma linguagem muito parecida com o LaTeX. Toda vez que você
quiser editar a documentação é necessário abrir esses arquivos e
modificá-los nesse formato. Para facilitar ainda mais a vida dos
programadores, existe o pacote `roxygen2`, que possibilita escrever a
documentação dentro da própria função `.R`, integrando código e
documentação em um único arquivo, e facilitando a edição das páginas de
ajuda. 

O pacote `roxygen2` pode ser utilizado também na abordagem tradicional
de criação de pacotes, eliminando uma etapa trabalhosa que é a de criar
os arquivos de documentação. No entanto, podemos usar todo o potencial
do `devtools` e do `roxygen2` juntos, e nos preocuparmos apenas com o
design das funções e não com todo o processo de criação do pacote em
si.

## Estrutura de um pacote

Basicamente um pacote do R é uma convenção para organizar e padronizar a
distribuição de funções extras do R. Todo pacote é composto
*obrigatoriamente* por apenas dois diretórios e dois arquivos de meta
dados:

* `R/`: um diretório contendo as funções em arquivos `*.R` (ex.:
  `foo.R`).
* `man/`: um diretório contendo a documentação (páginas de ajuda) de
  cada função do diretório acima. Os arquivos de documentação do R
  terminam com a extensão `.Rd` (ex.: `foo.Rd`).
* `DESCRIPTION`: um arquivo texto contendo as informações sobre o seu
  pacote: autor, licença, outros pacotes dependentes, ...
* `NAMESPACE`: um arquivo texto que informa quais funções do seu pacote
  serão exportadas, ou seja, aquelas que estarão disponíveis para o
  usuário, e quais funções são importadas de outros pacotes dos quais o
  seu depende.

Outros componentes que não são obrigatórios, mas podem estar presentes
no pacote são (nenhum deles serão abordados aqui):

* `tests/`: um diretório contendo scripts com testes unitários, rodados
  durante a criação do pacote, para testar se existe algum resultado não
  esperado sendo retornado por alguma de suas funções.
* `vignettes/`: um diretório que contém uma ou mais *vignettes*, que
  traduzidas literalmente são como "vinhetas", pequenos (ou mesmo
  grandes) textos que explicam com mais detalhes como os usuários podem
  usar as funções de seu pacote.
* `data/`: um diretório contendo arquivos de dados (normalmente em
  formato binário e comprimido do R, `.rda`), que podem ser usados como
  exemplos para aplicação das funções do pacote.

Caso você queira começar um pacote do início, as opções são: 1) criar
todos os componentes (diretórios e arquivos) manualmente e ir
alimentando com conteúdo, ou 2) usar a função `create()` do pacote
`devtools` para criar um "esqueleto" inicial com os arquivos e
diretórios fundamentais de um pacote. Portanto:


```r
library(devtools)
create("meupacote", rstudio = FALSE)
```

```
Creating package meupacote in .
No DESCRIPTION found. Creating with values:
```

```
Package: meupacote
Title: What the Package Does (one line, title case)
Version: 0.0.0.9000
Authors@R: person("First", "Last", email = "first.last@example.com", role = c("aut", "cre"))
Description: What the package does (one paragraph)
Depends: R (>= 3.2.1)
License: What license is it under?
LazyData: true
```

Como pode ser observado na saída acima, esse comando cria um diretório
chamado `meupacote`, contendo dois arquivos: `DESCRIPTION` e
`NAMESPACE`, e um diretório: `R/`


```
-rw-r--r-- DESCRIPTION
-rw-r--r-- NAMESPACE
drwxr-xr-x R
```

Note que na mensagem de criação acima, o arquivo `DESCRIPTION` já é
preenchido com alguns campos padrão, sendo necessário apenas alterá-los
com as informações do seu pacote.

Abaixo serão especificados os detalhes para a criação de cada um dos
componentes obrigatórios criados acima.

## `DESCRIPTION`: Caracterização do pacote

O arquivo `DESCRIPTION` é quem caracteriza o seu pacote: que é (são)
o(s) autor(es), uma breve descrição do que ele faz, qual o tipo de
licença, quais pacotes são necessários para que o seu funcione, e mais
alguns detalhes.

Seguindo os campos do `DESCRIPTION` criados anteriormente com a função
`create()`, temos alguns detalhes:

```
Package: meupacote
```
É simplesmente o nome do seu pacote.

```
Title: What the Package Does (one line, title case)
```
Uma breve descrição (de uma linha) sobre o que o seu pacote faz.

```
Version: 0.0.0.9000
```
A versão inicia (ou atual do pacote). O versionamento de
qualquer pedaço de software é uma coisa muito importante e não há um
consenso de que exista um padrão para todos os casos. No entanto, nos
pacotes do R, recomenda-se que as versões sejam do tipo `x.y-z` ou
`x.y.z` onde `x` seria a versão "maior", `y` a versão "menor", e `z`
os "patches" (alterações menores no código, como correção de bugs por
exemplo).

```
Authors@R: person("First", "Last", email = "first.last@example.com", role = c("aut", "cre"))
```
O autor, ou os autores e contribuidores do pacote. Este campo pode ser
preenchido com a função `person()` do R, no formato que está
apresentada. Se tiver mais de um autor, eles podem ser concatenados com
a função `c()`, por exemplo, `c(person(...), person(...))`. O argumento
`role` é importante pois é ele que identifica o papel de cada autor no
desenvolvimento do pacote. Por padrão, é necessário ter um (e somente
um) autor `"aut"` e um mantenedor `"cre"`, que pode ou não ser a mesma
pessoa. Outros papéis, como por exemplo o de um contribuidor
(`"ctb"`), e outros detalhes dessa função estão em `?person`.

```
Description: What the package does (one paragraph)
```
Uma descrição um pouco mais detalhada sobre o que o seu pacote faz. É
preciso ter pelo menos duas frases completas (sim, o R confere isso),
mas não muito maior do que um parágrafo.

```
Depends: R (>= 3.2.0)
```
Qual a versão do R que o seu pacote depende. É sempre prudente colocar
uma versão que seja maior ou igual a versão que você está desevolvendo o
pacote.

```
License: What license is it under?
```
A licença é fundamental se você pretende compartilhar o seu
pacote. Algumas opções comuns são: `CC0` (nenhum tipo de restrição),
`MIT` (prevê atribuição mas é bastante permissiva), e `GPL-3` (a mais
utilizada, requer que qualquer trabalho derivado do seu seja distribuido
com a mesma licença). Para mais opções e detalhes sobre licenças de
software livre, consulte [choosealicense.com][].

```
LazyData: true
```
Essa opção somente é importante se você pretende distribuir algum
conjunto de dados com o seu pacote (arquivos `.rda` no diretório
`data/`). Com a opção acima, os dados só serão carregados se realmente
forem chamados (com a função `data()`), e não carregados na
inicialização do pacote. Isso garante mais agilidade para carregar o
pacote e economiza memória caso os dados não sejam utilizados
(especialmente se as bases de dadpos forem gandes).

Outros campos do arquivo `DESCRIPTION` que serão necessários caso seu
pacote possua dependências:

* `Imports:` uma lista, separada por vígulas, dos pacotes dos quais o
  seu pacote precisa para funcionar. Estes são os pacotes que o seu
  depende, portanto eles também serão instalados (se já não estiverem),
  quando o seu for instalado. Colocar um pacote nesse campo garante que
  o pacote estará **instalado**, mas não que ele será carregado toda que
  vez que o seu pacote for carregado. Para utilizar funções de outros
  pacotes corretamente é necessário especifições da forma
  `pacote::funcao()` ou então utilizar o arquivo `NAMESPACE` conforme
  veremos abaixo.
* `Suggests:` os pacotes listados aqui não serão instalados junto com o
  seu. Eles podem ser usados, mas não são necessários. Por exemplo, você
  pode usar um pacote apenas para usar alguma base de dados ou apenas
  uma função. Use esse campo apenas se o seu pacote puder funcionar
  mesmo sem os pacotes especificados.

Para podermos utilizar esse arquivo para de fato gerarmos um pacote de
exemplo ao final deste tutorial, vamos modificar alguns campos e
utilizar o seguinte arquivo `DESCRIPTION`:



```
Package: meupacote
Title: Um Pacote de Exemplo
Version: 0.0-1
Authors@R: person("Fernando", "Mayer", email = "fernando.mayer@example.com", role = c("aut", "cre"))
Description: Este pacote serve apenas de tutorial. Mais informacoes
	     podem ser encontradas no texto do github.
Depends: R (>= 3.2.0)
License: GPL-3
LazyData: true
```

## `R/`: Funções

O diretório `R/` irá conter apenas as funções (arquivos `.R`) do seu
pacote. As funções podem estar em vários arquivos `.R` separados (um
para cada função) ou em um (ou alguns) arquivo único. A escolha é mais
uma questão de preferência pessoal, mas como veremos mais adiante,
utilizar arquivos separados para as funções torna o processo de
documentação um pouco mais ágil, além de facilitar a manutenção do
pacote com o tempo.

Para exemplificar, vamos criar uma função simples para fazer a soma de
dois números, e colocá-la dentro do diretório `R/` com o nome `soma.R`


```r
soma <- function(x, y){
    x + y
}
```

Com a função pronta, possivelmente iremos testá-la, e para isso vamos
utilizar a função `load_all()` do pacote `devtools`. Portanto, de dentro
de uma seção do R:


```r
load_all()
```

```
## Loading meupacote
```

irá carregar todas as funções que estiverem dentro do diretório `R/`,
tornado-as disponíveis para uso. Note que seria o mesmo resultado se
utilizassemos a função `source("soma.R")` para carregar a função, no
entanto, se tivermos várias funções, e/ou estivermos testando alterações
em muitas delas, a função `load_all()` é bem mais conveniente. Basta
fazer as alterações nas funções `.R`, carregá-las com `load_all()`e
testar novamente. Esse processo geralmente se repete muitas vezes
durante o desenvolvimento de um pacote.

## `man/`: Documentação

Você deve ter notado que o diretório `man/` não é criado da mesma forma
que os demais quando utilizamos a função `create()` acima. Isso porque
esse diretório depende das funções que serão criadas dentro do diretório
`R/`. No entanto, ele é também é obrigatório para podermos criar um
pacote mínimo.

Como mencionado anteriormente, a documentação de funções do R segue um
formato muito parecido com o LaTeX (mas com alguns comandos
próprios). Cada função criada dentro do diretório `R/`, independente de
estar em um arquivo separado ou único (com várias funções), deve
**obrigatoriamente** ter um arquivo correspondente dentro do diretório
`man/`, com a extensão `.Rd`. No caso da nossa função de exemplo acima,
que está em um arquivo chamado `soma.R`, precisaremos então de um
arquivo `soma.Rd` dentro do `man/`.

Existem duas formas de documentar uma função com os arquivos `.Rd`: uma
é da maneira tradicional, escrevendo manualmente a documentação
utilizando a linguagem específica e colocando os campos necessários. Os
detalhes de como fazer isso manualmente estão em
[Writing R documentation files][]. Outra forma de escrever a
documentação de uma função é utilizando o pacote `roxygen2`, que utiliza
algumas tags simples e permite que você escreva a documentação dentro do
próprio arquivo `.R`. Dessa forma fica muito mais simples alterar ou
atualizar a documentação de uma função, pois tudo se concentra dentro de
um único arquivo.

Neste texto vamos utilizar o pacote `roxygen2` para gerar a documentação
das funções. O primeiro passo é escrever a documentação dentro do
arquivo `.R`. A documentação usando o `roxygen2` segue o seguinte
formato:

* Toda documentação começa com um comentário especial, do tipo `#'`
  (note o `'` depois do `#`).
* Os campos de documentação são criados a parir de tags que iniciam com
  `@`, colocados logo após um comentário especial `#'`.
* Toda a documentação deve ficar diretamente acima do início da função.

Usando como exemplo a função `soma()` criada acima dentro do arquivo
`soma.R`, podemos documentá-la com o `roxygen2` da seguinte forma:



```
#' @title Faz a Soma de Dois Numeros
#' @name soma
#'
#' @description Uma (incrivel) funcao que pega dois numeros e faz a
#'     soma. Utilize este campo para descrever melhor o proposito de
#'     sua funcao e o que ela e capaz de fazer.
#'
#' @param x Um numero
#' @param y Outro numero
#'
#' @details Utilize este campo para escrever detalhes mais tecnicos da
#'     sua funcao (se necessario), ou para detalhar melhor como
#'     utilizar determinados argumentos.
#'
#' @return A soma dos numeros \code{x} e \code{y}.
#'
#' @author Fernando Mayer
#'
#' @seealso \code{\link[base]{sum}}, \code{\link[base]{+}}
#'
#' @examples
#' soma(2, 2)
#'
#' x <- 3
#' y <- 4
#' soma(x = x, y = y)
#'
#' @export
soma <- function(x, y){
    x + y
}
```

Para gerar a documentação agora basta utilizar a função `document()` do
pacote `devtools`:


```r
document()
```

```
## Updating meupacote documentation
## Loading meupacote
## First time using roxygen2 4.0. Upgrading automatically...
```

```
## Writing NAMESPACE
## Writing soma.Rd
```

> Observação: o comando `document()` do pacote `devtools` é de fato um
> "wrapper" para a função `roxygenize()` do pacote `roxygen2`.

O resultado da chamada dessa função é:

* Criar o diretório `man/` se ele ainda não existir, ou se for a
  primeira vez que estiver criando a documentação com a função
  `document()`.
* Gerar os arquivos `.Rd` dentro de `man/` (se ainda não existirem)
  correspondentes às funções no diretório `.R`.
* Se os arquivos `.Rd` já existirem, a chamada da função `document()`
  irá apenas atualizar os arquivos que foram modificados.
* Escrever no arquivo `NAMESPACE` as funções a serem exportadas ou
  importadas. Veremos mais detalhes abaixo na seção
  [`NAMESPACE`][6].

No nosso exemplo, a chamada da função `document()` criou o diretório
`man/` e gerou o arquivo `soma.Rd` com o seguinte conteúdo


```
% Generated by roxygen2 (4.1.1): do not edit by hand
% Please edit documentation in R/soma.R
\name{soma}
\alias{soma}
\title{Faz a Soma de Dois Numeros}
\usage{
soma(x, y)
}
\arguments{
\item{x}{Um numero}

\item{y}{Outro numero}
}
\value{
A soma dos numeros \code{x} e \code{y}.
}
\description{
Uma (incrivel) funcao que pega dois numeros e faz a
    soma. Utilize este campo para descrever melhor o proposito de
    sua funcao e o que ela e capaz de fazer.
}
\details{
Utilize este campo para escrever detalhes mais tecnicos da
    sua funcao (se necessario), ou para detalhar melhor como
    utilizar determinados argumentos.
}
\examples{
soma(2, 2)

x <- 3
y <- 4
soma(x = x, y = y)
}
\author{
Fernando Mayer
}
\seealso{
\code{\link[base]{sum}}, \code{\link[base]{+}}
}
```

Note que é muito mais simples ecrever com o `roxygen2` do que ter que
editar esse arquivo todo a mão. Note também o comentário no início desse
arquivo:


```
% Generated by roxygen2 (4.1.1): do not edit by hand
% Please edit documentation in R/soma.R
```

O `roxygen2` deixa claro que você não deve (e não precisa) modificar
esse arquivo `soma.Rd` para atualizar a documentação. Sempre que alterar
alguma coisa faça no próprio arquivo `.R`, e a função `document()` irá
atualizar a documentação no arquivo `.Rd`.

Para conferir se a documentação está de acordo com o que você espera,
você pode conferir a qualquer momento, por exemplo, com `?soma` ou
`help(soma)`, após rodar a função `load_all()` para carregar a
documentação atualizada depois de `document()`.

Uma outra forma mais automática de conferir se a documentação está
escrita de maneira correta, é utilizando a função `check_doc()` do
pacote `devtools`:


```r
check_doc()
```

```
## Updating meupacote documentation
## Loading meupacote
## Checking documentation
```

Se nenhuma mensagem de aviso ou erro apareceu acima, então a
documentação está de acordo com o esperado.

Alguns detalhes das tags utilizadas nesse exemplo do `roxygen2`:

* `@title`: um título curto da função (deve ser capitalizado).
* `@name`: o nome da função.
* `@description`: descreve o que a função faz e pra que ela serve, de
  maneira geral.
* `@param`: descreve o que cada argumento da função faz. Deve ser sempre
  no formato: `@param` <kbd>SPC</kbd> `argumento` <kbd>SPC</kbd>
  `descrição do argumento`.
* `@details`: espaço para escrever mais detalhes, geralmente mais
  técnicos, sobre a sua função, e/ou para detalhar alguma coisa
  específica de algum argumento.
* `@return`: especifica o que a sua função retorna (um gráfico, uma
  lista, um número, ...)
* `@author`: autr(es) da função.
* `@seealso`: outras funções que podem ser úteis para quem está usando a
  sua função, ou funções similares. No exemplo acima temos duas
  marcações que são específicas da documentação do R: `\code{}` para que
  o texto dentro das chaves saia com fonte fixa (`como esta`),
  geralmente para especificar nomes de funções, etc.;
  `\link[<pacote>]{<função>}` faz o link para a página de ajuda de
  alguma `<função>` dentro de algum `<pacote>`. Mais marcações do
  formato `.Rd` podem ser encontradas no manual de documentação do R
  (seção [Rd format][]), e podem ser utilizadas dentro na documentação
  com `roxygen2`.
* `@example`: exemplos de uso da sua função.
* `@export`: esta tag é a responsável por sua função ficar disponível
  para os usuários, e é o que faz a função ser incluida no `NAMESPACE`
  (mais detalhes serão vistos na seção
  [`NAMESPACE`][6] abaixo). Se você não quer que a
  função seja disponibilizada para os usuários quando carregarem o seu
  pacote (em funções internas ou auxiliares por exemplo), simplesmente
  não inclua esta tag na documentação.

## `NAMESPACE`: Organização

O `NAMESPACE` é um arquivo que ajuda a organizar as funções exportadas e
importadas pelo seu pacote. Ele ajuda um pacote a ser auto contido, no
sentido de que ele não vai interferir no funcionamento de outros
pacotes, e que também os outros pacotes não irão interferir no
funcionamento do seu.

Normalmente, a especificação do `NAMESPACE` é a parte mais difícil da
criação de um pacote, mas com o uso do `devtools` como estamos fazendo,
normalmente não será necessário se preocupar com todos os detalhes que
envolvem esse arquivo. O mais importante é saber quais funções serão
exportadas do seu pacote, quais serão importadas, e de que maneira
podemos especificar essas funções.

### Exportando funções

O conceito de exportar uma ou mais funções de um pacote, é o de tornar
estas funções disponíveis para o usuário. No desenvolvimento de um
pacote, normalmente são criadas algumas funções internas ou auxiliares,
que são utilizadas por uma ou mais funções principais. Se em nenhum
momento estas funções precisarem ser utilizadas pelo usuário do pacote,
então elas não precisam (e não devem) ser exportadas, para minimizar a
possibilidade de conflito com outros pacotes.

Quando um pacote é carregado com `library()` ou `require()`, ele é
"anexado" (*attached*) ao workspace do usuário, tornando assim as
funções exportadas disponíveis para serem utilizadas. Você pode conferir
a lista de pacotes anexados em seu workspace com `search()`.

Por padrão, quando iniciamos um pacote com a função `create()` do pacote
`devtools`, é criado um arquivo `NAMESPACE` que exporta automaticamente
**todas** as funções que forem criadas, e que não começem com um ponto
`.`. O arquivo `NAMESPACE` inicial possui essa especificação:


```
exportPattern("^[^\\.]")
```

No entanto, se você não quiser exportar todas as funções de dentro do
diretório `R/`, será necessário especificar quais funções deseja
exportar. Para isso, basta usar a tag `@export` na documentação da
função com o `roxygen2` (assim como usamos na documentação da função
`soma.R` acima). A função `document()` é a responsável por verificar as
funções que possuem `@export` e colocá-las adequadamente no arquivo
`NAMESPACE`. Dessa forma, usando o nosso exemplo, após escrever a função
`soma.R` com `@export` e rodar `document()`, o arquivo `NAMESPACE` agora
será:


```
# Generated by roxygen2 (4.1.1): do not edit by hand

export(soma)
```

Se houverem mais funções exportadas, elas aparecerão nesse arquivo, uma
em cada linha. Caso existam funções que você não queira exportar, basta
não colocar a tag `@export` na documentação da função. (Na verdade, as
funções não exportadas não precisam nem ser documentadas, mas é sempre
uma boa prática escrever a documentação até mesmo para você no futuro).

### Importando funções

Normalmente as funções que estamos criando para um pacote utilizam
funções de outros pacotes. Todas as funções dos pacotes que são
distribuídos e automaticamente carregados quando você inicia uma sessão
do R, são disponíveis para serem utilizadas em qualquer outra função,
sem a necessidade de qualquer especificação adicional. Estes pacotes
básicos do R são: `base`, `utils`, `datasets`, `grDevices`, `graphics`,
`stats`, e `methods`. Esse conjunto de pacotes já garante uma grande
quantidade de funções, no entanto, algumas vezes podem ser necessárias
funções específicas de outros pacotes.

Existem três formas básicas de importar, ou seja, utilizar funções de
outros pacotes dentro do seu pacote:

1. `pacote::funcao()`: chamar uma função utilizando o operador `::` é
   recomendado quando o função for utilizada poucas vezes. Dessa forma
   fica claro no seu pacote aonde uma função externa está sendo
   utilizada.
2. `@importFrom pacote funcao`: se uma funcao for utilizada váras vezes
   dentro de um pacote, ou se mais de uma função do mesmo pacote for
   utilizada, recomenda-se especifica-las com a tag `@importFrom` na
   documentação `roxygen2`. Dessa forma não é necessário utilizar o
   operador `::` na chamada das funções, o que torna o código um pouco
   mais legível se muitas funções externas forem utilizadas.
3. `@import pacote`: se o seu pacote depende de muitas funções de
   outro(s) pacote(s), então o mais razoável é importar todas as funções
   do pacote. Isso é feito utilizando a tag `@import` na documentação do
   `roxygen2` da função. Dessa forma, todas as funções do pacote estarão
   disponíveis para serem utilizadas no seu pacote.

Já vimos anteriromemte na seção [`DESCRIPTION`][3] que exite um campo
`Imports` que serve para especificar quais pacotes o seu depende. Se
você utilizou qualquer um dos três métodos acima para importar uma ou
várias funções, então **obrigatoriamente** o pacote que contém estas
funções deve ser listado no campo `Imports` do arquivo `DESCRIPTION`.

Uma distinção importante é que listar os pacotes no campo `Imports` do
arquivo `DESCRIPTION`, garante que o usuário terá instalado os pacotes
necessários para que o seu funcione, mas ele não é o responsável por
carregar e tornar as funções desse pacote disponível. Esse papel é feito
por algum dos três métodos citados acima, que por consequência irão
atualizar o arquivo `NAMESPACE`, que é o verdadeiro responsável por
carregar os pacotes e funções necessárias para o uso do seu pacote.

Por exemplo, vamos criar uma função personalizada para fazer um gráfico
de pontos utilizando a função `xyplot()` do pacote `lattice`. Vamos
supor que esse gráfico personalizado usa um `x` no lugar do ponto (`pch
= 4`), e de cor preto (`col = 1`), já que o padrão do `lattice` é azul
claro. Vamos chamar essa função de `meuxy()` e colocá-la no arquivo
`meuxy.R` do diretório `R/`. O conteúdo desse arquivo é:



```
#' @title Faz um Grafico Personalizado
#' @name meuxy
#'
#' @description Uma (incrivel) funcao para fazer um xyplot
#'     personalizado. Usando o pacote lattice.
#'
#' @param x Uma formula do lattice
#' @param ... Outros argumentos passados para a funcao
#'     \code{\link[lattice]{xyplot}}
#'
#' @details Utilize este campo para escrever detalhes mais tecnicos da
#'     sua funcao (se necessario), ou para detalhar melhor como
#'     utilizar determinados argumentos.
#'
#' @return Um grafico xyplot do lattice com \code{pch = 4} e \code{col =
#'     1}.
#'
#' @author Fernando Mayer
#'
#' @seealso \code{\link[lattice]{xyplot}}
#'
#' @examples
#' meuxy(1:10 ~ 1:10)
#'
#' @importFrom lattice xyplot
#'
#' @export
meuxy <- function(x, ...){
    xyplot(x, pch = 4, col = 1, ...)
}
```

Agora é necessário rodar a função `document()` para gerar o arquivo
`man/meuxy.Rd` e atualizar o arquivo `NAMESPACE`, com a exportação da
função `meuxy()`, e a importação da função `xyplot()` do pacote
`lattice`. 


```r
document()
```

```
## Updating meupacote documentation
## Loading meupacote
```

```
## Writing NAMESPACE
## Writing meuxy.Rd
```

Note que, na mensagem acima, além de criar o arquivo `meuxy.Rd`, houve
uma atualização do arquivo `NAMESPACE`, que agora irá conter:


```
# Generated by roxygen2 (4.1.1): do not edit by hand

export(meuxy)
export(soma)
importFrom(lattice,xyplot)
```
Ou seja, já estamos exportando a função `meuxy()`, e importando a função
`xyplot()` do pacote `lattice`, através da função `importFrom()`,
colocada no `NAMESPACE`


```
importFrom(lattice,xyplot)
```

Note que, neste exemplo, utilizamos a tag


```r
#' @importFrom lattice xyplot
```

Mas poderíamos também ter utilizado o operador `::` diretamente na
função, por exemplo


```r
meuxy <- function(x, ...){
    lattice::xyplot(x, pch = 4, col = 1, ...)
}
```

E, nesse caso, não precisariamos usar a tag `@importFrom`. O resultado
no `NAMESPACE` seria o mesmo. A tag `@import` também poderia ter sido
utilizada, por exemplo,


```r
#' @import lattice
```
No entanto, nesse caso não há vantagem em importar todo o pacote pois
estamos utilizando apenas uma função.


## Workflow



## Extra: integração contínua com Travis-CI



[1]: #motivação
[2]: #estrutura-de-um-pacote
[3]: #description-caracterização-do-pacote
[4]: #r-funções
[5]: #man-documentação
[6]: #namespace-organização
[6.1]: #exportando-funções
[6.2]: #importando-funções
[7]: #workflow
[8]: #extra-integração-contínua-com-travis-ci
[John M. Chambers]: http://statweb.stanford.edu/~jmc4/
[choosealicense.com]: http://choosealicense.com
[Writing R documentation files]: [http://cran.r-project.org/doc/manuals/r-release/R-exts.html#Writing-R-documentation-files]
[Rd format]: http://cran.r-project.org/doc/manuals/r-release/R-exts.html#Rd-format
