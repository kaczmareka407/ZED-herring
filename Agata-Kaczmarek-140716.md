---
title: "Stopniowe karłowacenie śledzi"
author: "Agata Kaczmarek 140716"
date: "2022-12-10"
output: 
  html_document:
    toc: true
    toc_float: true
    number_sections: yes
    keep_md: yes
    code_folding: hide
---



# Podsumowanie analizy

W raporcie badano problem malejącej długości śledzia. Wczytano, opisano i oczyszczono dane. Dokonano analizy danych i koorelacji atrybutów. Zauważono, że największy wpływ przy koorelacji na długość śledzia ma temperatura przy powierzchni wody. Podczas wykorzystania regresorów dla regresji liniowej również ajwiększy wpływ maiała temperatura wody, a dla algorytmu Random Forrest natężenie połowów w regionie. Dla regresji liniowej przewidywana długość śledzia wynosi od 22.75cm 32.33cm, a dla Random Forrest od 20.82cm do 28.41cm. 

# Wykorzystane biblioteki

Do przygotowania raportu wykorzystano bilbioteki:


- caret
- data.table
- dplyr
- ggcorrplot
- ggplot2
- kableExtra
- knitr
- plotly
- tidyr




# Opis problemu 

W Europie zauważono stopniowy spadek długości śledzia oceanicznego, dlatego zbadano warunki w jakich żyją oraz zmierzono ich długość. Obserwacje (50-100 trzyletnich śledzi), które odbywały się w połowach komercyjnych jednostek dotyczą ostatnich 60 lat. 

# Wczytanie danych

Wczytano plik podany na platformie eKursy oraz wyświetlono pierwsze oraz ostatnie wiersze.


```r
read.csv(file='sledzie.csv')->herring_clear

kable(head(herring_clear[,1:8]), format = "markdown", caption = "Herring - początek pobranego zbioru danych PART 1")
```



Table: Herring - początek pobranego zbioru danych PART 1

|  X| length|cfin1   |cfin2   |chel1   |chel2    |lcop1   |lcop2    |
|--:|------:|:-------|:-------|:-------|:--------|:-------|:--------|
|  0|   23.0|0.02778 |0.27785 |2.46875 |?        |2.54787 |26.35881 |
|  1|   22.5|0.02778 |0.27785 |2.46875 |21.43548 |2.54787 |26.35881 |
|  2|   25.0|0.02778 |0.27785 |2.46875 |21.43548 |2.54787 |26.35881 |
|  3|   25.5|0.02778 |0.27785 |2.46875 |21.43548 |2.54787 |26.35881 |
|  4|   24.0|0.02778 |0.27785 |2.46875 |21.43548 |2.54787 |26.35881 |
|  5|   22.0|0.02778 |0.27785 |2.46875 |21.43548 |2.54787 |?        |

```r
kable(head(herring_clear[,9:16]), format = "markdown", caption = "Herring - początek pobranego zbioru danych PART 2")
```



Table: Herring - początek pobranego zbioru danych PART 2

|  fbar|   recr|      cumf|   totaln|sst           |      sal| xmonth| nao|
|-----:|------:|---------:|--------:|:-------------|--------:|------:|---:|
| 0.356| 482831| 0.3059879| 267380.8|14.3069330186 | 35.51234|      7| 2.8|
| 0.356| 482831| 0.3059879| 267380.8|14.3069330186 | 35.51234|      7| 2.8|
| 0.356| 482831| 0.3059879| 267380.8|14.3069330186 | 35.51234|      7| 2.8|
| 0.356| 482831| 0.3059879| 267380.8|14.3069330186 | 35.51234|      7| 2.8|
| 0.356| 482831| 0.3059879| 267380.8|14.3069330186 | 35.51234|      7| 2.8|
| 0.356| 482831| 0.3059879| 267380.8|14.3069330186 | 35.51234|      7| 2.8|

```r
kable(tail(herring_clear[,1:8]), format = "markdown", caption = "Herring - koniec pobranego zbioru danych PART 1")
```



Table: Herring - koniec pobranego zbioru danych PART 1

|      |     X| length|cfin1   |cfin2   |chel1   |chel2    |lcop1    |lcop2    |
|:-----|-----:|------:|:-------|:-------|:-------|:--------|:--------|:--------|
|52577 | 52576|   21.5|0       |0.01    |1.02143 |26.00617 |1.06429  |34.1456  |
|52578 | 52577|   24.0|1.02508 |3.66319 |6.42127 |25.51806 |10.92857 |37.39201 |
|52579 | 52578|   26.0|1.02508 |3.66319 |6.42127 |25.51806 |10.92857 |37.39201 |
|52580 | 52579|   25.0|1.02508 |3.66319 |6.42127 |25.51806 |10.92857 |37.39201 |
|52581 | 52580|   25.0|0.36032 |5.36402 |4.32674 |27.16006 |5.08099  |36.6877  |
|52582 | 52581|   23.5|0.36032 |5.36402 |4.32674 |27.16006 |?        |36.6877  |

```r
kable(tail(herring_clear[,9:16]), format = "markdown", caption = "Herring - koniec pobranego zbioru danych PART 2")
```



Table: Herring - koniec pobranego zbioru danych PART 2

|      |  fbar|    recr|      cumf|   totaln|sst           |      sal| xmonth|   nao|
|:-----|-----:|-------:|---------:|--------:|:-------------|--------:|------:|-----:|
|52577 | 0.100| 1322000| 0.0922202| 648314.9|14.5555996798 | 35.53620|      7|  2.05|
|52578 | 0.485|  724151| 0.3838187| 457143.9|13.7115996983 | 35.51169|     11|  2.05|
|52579 | 0.485|  724151| 0.3838187| 457143.9|13.7115996983 | 35.51169|     11|  2.05|
|52580 | 0.485|  724151| 0.3838187| 457143.9|13.7115996983 | 35.51169|     11|  2.05|
|52581 | 0.434|  441827| 0.3726272| 191976.2|14.4795996814 | 35.50777|      6| -1.90|
|52582 | 0.434|  441827| 0.3726272| 191976.2|14.4795996814 | 35.50777|      6| -1.90|


# Przetwarzanie danych

Sprawdzono, w których kolumnach pojawiają się wartości ?. Jeżeli taka wartość wystąpiła to była pobierana wartość z poprzedniego wiersza dla danej kolumny i przypisywana do sprawdzanego. W przypadku pustych wartości w pierwszym wierszu dane były pobierane z drugiego wiersza. Dodatkowo sprawdzono typy danych dla kolumn. Kolumny, które były typem `character` zamieniono na `numeric`. Dane są podane chronologicznie według zapisanych obserwacji. 



```r
herring_clear->herring
herring[herring == '?'] <- NA


herring<-fill(herring,cfin1,cfin2,chel1, chel2, lcop1, lcop2, sst, .direction ="updown")
print(sapply(herring, class)) 
```

```
##           X      length       cfin1       cfin2       chel1       chel2 
##   "integer"   "numeric" "character" "character" "character" "character" 
##       lcop1       lcop2        fbar        recr        cumf      totaln 
## "character" "character"   "numeric"   "integer"   "numeric"   "numeric" 
##         sst         sal      xmonth         nao 
## "character"   "numeric"   "integer"   "numeric"
```

```r
herring[, 3:8] <- sapply(herring[, 3:8], as.numeric)
herring[, 13] <- sapply(herring[, 13], as.numeric)
print(sapply(herring, class)) 
```

```
##         X    length     cfin1     cfin2     chel1     chel2     lcop1     lcop2 
## "integer" "numeric" "numeric" "numeric" "numeric" "numeric" "numeric" "numeric" 
##      fbar      recr      cumf    totaln       sst       sal    xmonth       nao 
## "numeric" "integer" "numeric" "numeric" "numeric" "numeric" "integer" "numeric"
```

```r
kable(head(herring[,1:8]), format = "markdown", digits = 2, caption = "Herring - początek sformatowanego zbioru danych PART 1")
```



Table: Herring - początek sformatowanego zbioru danych PART 1

|  X| length| cfin1| cfin2| chel1| chel2| lcop1| lcop2|
|--:|------:|-----:|-----:|-----:|-----:|-----:|-----:|
|  0|   23.0|  0.03|  0.28|  2.47| 21.44|  2.55| 26.36|
|  1|   22.5|  0.03|  0.28|  2.47| 21.44|  2.55| 26.36|
|  2|   25.0|  0.03|  0.28|  2.47| 21.44|  2.55| 26.36|
|  3|   25.5|  0.03|  0.28|  2.47| 21.44|  2.55| 26.36|
|  4|   24.0|  0.03|  0.28|  2.47| 21.44|  2.55| 26.36|
|  5|   22.0|  0.03|  0.28|  2.47| 21.44|  2.55| 26.36|

```r
kable(head(herring[,9:16]), format = "markdown", digits = 2, caption = "Herring - początek sformatowanego zbioru danych PART 2")
```



Table: Herring - początek sformatowanego zbioru danych PART 2

| fbar|   recr| cumf|   totaln|   sst|   sal| xmonth| nao|
|----:|------:|----:|--------:|-----:|-----:|------:|---:|
| 0.36| 482831| 0.31| 267380.8| 14.31| 35.51|      7| 2.8|
| 0.36| 482831| 0.31| 267380.8| 14.31| 35.51|      7| 2.8|
| 0.36| 482831| 0.31| 267380.8| 14.31| 35.51|      7| 2.8|
| 0.36| 482831| 0.31| 267380.8| 14.31| 35.51|      7| 2.8|
| 0.36| 482831| 0.31| 267380.8| 14.31| 35.51|      7| 2.8|
| 0.36| 482831| 0.31| 267380.8| 14.31| 35.51|      7| 2.8|


# Rozmiar zbioru i statystyki

Wszystkie wartości w zbiorze są numeric lub integer. Dotyczą one:

- *length*: długość złowionego śledzia [cm];

- cfin1: dostępnośś planktonu [zagęszczenie Calanus finmarchicus gat. 1];
- cfin2: dostępność planktonu [zagęszczenie Calanus finmarchicus gat. 2];
- chel1: dostępność planktonu [zagęszczenie Calanus helgolandicus gat. 1];
- chel2: dostępność planktonu [zagęszczenie Calanus helgolandicus gat. 2];
- lcop1: dostępność planktonu [zagęszczenie widłonogów gat. 1];
- lcop2: dostępność planktonu [zagęszczenie widłonogów gat. 2];

- fbar: natężenie połowów w regionie [ułamek pozostawionego narybku];
- recr: roczny narybek [liczba śledzi];
- cumf: łączne roczne natężenie połowów w regionie [ułamek pozostawionego narybku];
- totaln: łączna liczba ryb złowionych w ramach połowu [liczba śledzi];

- sst: temperatura przy powierzchni wody [°C];
- sal: poziom zasolenia wody [Knudsen ppt];
- xmonth: miesiąc połowu [numer miesiąca];
- nao: oscylacja północnoatlantycka [mb].

Zbiór danych zawiera 52582 wierszy i 16 kolumn.
Minimalna długość śledzia: 19.
Maksymalna długość śledzia: 32.5.
Występuje 0 różnych długości.


Poniżej zostało przedstawione podsumowanie dotyczące wszystkich atrybutów.



```r
kable(summary(herring)[,1:5])
```

<table>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;">       X </th>
   <th style="text-align:left;">     length </th>
   <th style="text-align:left;">     cfin1 </th>
   <th style="text-align:left;">     cfin2 </th>
   <th style="text-align:left;">     chel1 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Min.   :    0 </td>
   <td style="text-align:left;"> Min.   :19.0 </td>
   <td style="text-align:left;"> Min.   : 0.0000 </td>
   <td style="text-align:left;"> Min.   : 0.0000 </td>
   <td style="text-align:left;"> Min.   : 0.000 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 1st Qu.:13145 </td>
   <td style="text-align:left;"> 1st Qu.:24.0 </td>
   <td style="text-align:left;"> 1st Qu.: 0.0000 </td>
   <td style="text-align:left;"> 1st Qu.: 0.2778 </td>
   <td style="text-align:left;"> 1st Qu.: 2.469 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Median :26291 </td>
   <td style="text-align:left;"> Median :25.5 </td>
   <td style="text-align:left;"> Median : 0.1111 </td>
   <td style="text-align:left;"> Median : 0.7012 </td>
   <td style="text-align:left;"> Median : 5.750 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Mean   :26291 </td>
   <td style="text-align:left;"> Mean   :25.3 </td>
   <td style="text-align:left;"> Mean   : 0.4457 </td>
   <td style="text-align:left;"> Mean   : 2.0255 </td>
   <td style="text-align:left;"> Mean   :10.003 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 3rd Qu.:39436 </td>
   <td style="text-align:left;"> 3rd Qu.:26.5 </td>
   <td style="text-align:left;"> 3rd Qu.: 0.3333 </td>
   <td style="text-align:left;"> 3rd Qu.: 1.7936 </td>
   <td style="text-align:left;"> 3rd Qu.:11.500 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Max.   :52581 </td>
   <td style="text-align:left;"> Max.   :32.5 </td>
   <td style="text-align:left;"> Max.   :37.6667 </td>
   <td style="text-align:left;"> Max.   :19.3958 </td>
   <td style="text-align:left;"> Max.   :75.000 </td>
  </tr>
</tbody>
</table>

```r
kable(summary(herring)[,6:11])
```

<table>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;">     chel2 </th>
   <th style="text-align:left;">     lcop1 </th>
   <th style="text-align:left;">     lcop2 </th>
   <th style="text-align:left;">      fbar </th>
   <th style="text-align:left;">      recr </th>
   <th style="text-align:left;">      cumf </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Min.   : 5.238 </td>
   <td style="text-align:left;"> Min.   :  0.3074 </td>
   <td style="text-align:left;"> Min.   : 7.849 </td>
   <td style="text-align:left;"> Min.   :0.0680 </td>
   <td style="text-align:left;"> Min.   : 140515 </td>
   <td style="text-align:left;"> Min.   :0.06833 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 1st Qu.:13.427 </td>
   <td style="text-align:left;"> 1st Qu.:  2.5479 </td>
   <td style="text-align:left;"> 1st Qu.:17.808 </td>
   <td style="text-align:left;"> 1st Qu.:0.2270 </td>
   <td style="text-align:left;"> 1st Qu.: 360061 </td>
   <td style="text-align:left;"> 1st Qu.:0.14809 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Median :21.435 </td>
   <td style="text-align:left;"> Median :  7.0000 </td>
   <td style="text-align:left;"> Median :24.859 </td>
   <td style="text-align:left;"> Median :0.3320 </td>
   <td style="text-align:left;"> Median : 421391 </td>
   <td style="text-align:left;"> Median :0.23191 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Mean   :21.215 </td>
   <td style="text-align:left;"> Mean   : 12.8079 </td>
   <td style="text-align:left;"> Mean   :28.419 </td>
   <td style="text-align:left;"> Mean   :0.3304 </td>
   <td style="text-align:left;"> Mean   : 520367 </td>
   <td style="text-align:left;"> Mean   :0.22981 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 3rd Qu.:27.193 </td>
   <td style="text-align:left;"> 3rd Qu.: 21.2315 </td>
   <td style="text-align:left;"> 3rd Qu.:37.232 </td>
   <td style="text-align:left;"> 3rd Qu.:0.4560 </td>
   <td style="text-align:left;"> 3rd Qu.: 724151 </td>
   <td style="text-align:left;"> 3rd Qu.:0.29803 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Max.   :57.706 </td>
   <td style="text-align:left;"> Max.   :115.5833 </td>
   <td style="text-align:left;"> Max.   :68.736 </td>
   <td style="text-align:left;"> Max.   :0.8490 </td>
   <td style="text-align:left;"> Max.   :1565890 </td>
   <td style="text-align:left;"> Max.   :0.39801 </td>
  </tr>
</tbody>
</table>


Z wcześniej wspomnianych założeń, że badania są podane chronologicznie można zobaczyć jak na początku rosła długość śledzia, a później częściej malała. Można to zauważyć przy 16640-tej obserwacji.


```r
p<-ggplot(herring, aes(x=X)) + 
  geom_smooth(aes(y = length,colour="lenght"), color = "#4477AA") +
  ggtitle("Animacja długości śledzia") +
  xlab("Numer obserwacji") +
  ylab("Długość śledzia [cm]")

ggplotly(p)
```

```{=html}
<div id="htmlwidget-e5517864d16dd925dc45" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-e5517864d16dd925dc45">{"x":{"data":[{"x":[0,665.582278481013,1331.16455696203,1996.74683544304,2662.32911392405,3327.91139240506,3993.49367088608,4659.07594936709,5324.6582278481,5990.24050632911,6655.82278481013,7321.40506329114,7986.98734177215,8652.56962025316,9318.15189873418,9983.73417721519,10649.3164556962,11314.8987341772,11980.4810126582,12646.0632911392,13311.6455696203,13977.2278481013,14642.8101265823,15308.3924050633,15973.9746835443,16639.5569620253,17305.1392405063,17970.7215189873,18636.3037974684,19301.8860759494,19967.4683544304,20633.0506329114,21298.6329113924,21964.2151898734,22629.7974683544,23295.3797468354,23960.9620253165,24626.5443037975,25292.1265822785,25957.7088607595,26623.2911392405,27288.8734177215,27954.4556962025,28620.0379746835,29285.6202531646,29951.2025316456,30616.7848101266,31282.3670886076,31947.9493670886,32613.5316455696,33279.1139240506,33944.6962025316,34610.2784810127,35275.8607594937,35941.4430379747,36607.0253164557,37272.6075949367,37938.1898734177,38603.7721518987,39269.3544303797,39934.9367088608,40600.5189873418,41266.1012658228,41931.6835443038,42597.2658227848,43262.8481012658,43928.4303797468,44594.0126582278,45259.5949367089,45925.1772151899,46590.7594936709,47256.3417721519,47921.9240506329,48587.5063291139,49253.0886075949,49918.6708860759,50584.253164557,51249.835443038,51915.417721519,52581],"y":[24.4010153612675,24.5303404158547,24.658594572464,24.7847069331177,24.9076065998379,25.0262226746469,25.1394842595668,25.2463204566198,25.3456603678281,25.4364397962695,25.5186826289461,25.5946944623356,25.6670682006799,25.7383967482213,25.8112730092018,25.8882898878635,25.9720402884484,26.0651171151988,26.1699763559562,26.2849215808547,26.4034557287372,26.5188143236023,26.6242328894484,26.7129469502737,26.7781920300768,26.813203652856,26.8112173426098,26.7662782009043,26.6818426685312,26.5674390180405,26.4326967191782,26.2872452416902,26.1407140553225,26.0027326298212,25.8829304349321,25.7909369404013,25.7343807691038,25.7096943988256,25.7094180349241,25.7260879748526,25.7522405160644,25.7804119560128,25.8031385921511,25.8129567219325,25.8024053429259,25.7667127677505,25.7088431399685,25.6331430622843,25.5439591374027,25.445637968028,25.3425261568649,25.238970306618,25.1393170199917,25.0478741652679,24.9666255453584,24.893952661852,24.827927140955,24.7666206088735,24.7081046918135,24.6504510159814,24.5917312075832,24.5300168928251,24.4634982605659,24.3924939364037,24.319163586806,24.2457275823187,24.1744062934876,24.1074200908583,24.0469893449766,23.9953344263883,23.9546757056391,23.9269369557118,23.9116864634045,23.9073692495261,23.9124234171583,23.9252870693826,23.9443983092805,23.9681952399337,23.9951159644238,24.0235985858323],"text":["X:     0.0000<br />length: 24.40102<br />colour: #4477AA","X:   665.5823<br />length: 24.53034<br />colour: #4477AA","X:  1331.1646<br />length: 24.65859<br />colour: #4477AA","X:  1996.7468<br />length: 24.78471<br />colour: #4477AA","X:  2662.3291<br />length: 24.90761<br />colour: #4477AA","X:  3327.9114<br />length: 25.02622<br />colour: #4477AA","X:  3993.4937<br />length: 25.13948<br />colour: #4477AA","X:  4659.0759<br />length: 25.24632<br />colour: #4477AA","X:  5324.6582<br />length: 25.34566<br />colour: #4477AA","X:  5990.2405<br />length: 25.43644<br />colour: #4477AA","X:  6655.8228<br />length: 25.51868<br />colour: #4477AA","X:  7321.4051<br />length: 25.59469<br />colour: #4477AA","X:  7986.9873<br />length: 25.66707<br />colour: #4477AA","X:  8652.5696<br />length: 25.73840<br />colour: #4477AA","X:  9318.1519<br />length: 25.81127<br />colour: #4477AA","X:  9983.7342<br />length: 25.88829<br />colour: #4477AA","X: 10649.3165<br />length: 25.97204<br />colour: #4477AA","X: 11314.8987<br />length: 26.06512<br />colour: #4477AA","X: 11980.4810<br />length: 26.16998<br />colour: #4477AA","X: 12646.0633<br />length: 26.28492<br />colour: #4477AA","X: 13311.6456<br />length: 26.40346<br />colour: #4477AA","X: 13977.2278<br />length: 26.51881<br />colour: #4477AA","X: 14642.8101<br />length: 26.62423<br />colour: #4477AA","X: 15308.3924<br />length: 26.71295<br />colour: #4477AA","X: 15973.9747<br />length: 26.77819<br />colour: #4477AA","X: 16639.5570<br />length: 26.81320<br />colour: #4477AA","X: 17305.1392<br />length: 26.81122<br />colour: #4477AA","X: 17970.7215<br />length: 26.76628<br />colour: #4477AA","X: 18636.3038<br />length: 26.68184<br />colour: #4477AA","X: 19301.8861<br />length: 26.56744<br />colour: #4477AA","X: 19967.4684<br />length: 26.43270<br />colour: #4477AA","X: 20633.0506<br />length: 26.28725<br />colour: #4477AA","X: 21298.6329<br />length: 26.14071<br />colour: #4477AA","X: 21964.2152<br />length: 26.00273<br />colour: #4477AA","X: 22629.7975<br />length: 25.88293<br />colour: #4477AA","X: 23295.3797<br />length: 25.79094<br />colour: #4477AA","X: 23960.9620<br />length: 25.73438<br />colour: #4477AA","X: 24626.5443<br />length: 25.70969<br />colour: #4477AA","X: 25292.1266<br />length: 25.70942<br />colour: #4477AA","X: 25957.7089<br />length: 25.72609<br />colour: #4477AA","X: 26623.2911<br />length: 25.75224<br />colour: #4477AA","X: 27288.8734<br />length: 25.78041<br />colour: #4477AA","X: 27954.4557<br />length: 25.80314<br />colour: #4477AA","X: 28620.0380<br />length: 25.81296<br />colour: #4477AA","X: 29285.6203<br />length: 25.80241<br />colour: #4477AA","X: 29951.2025<br />length: 25.76671<br />colour: #4477AA","X: 30616.7848<br />length: 25.70884<br />colour: #4477AA","X: 31282.3671<br />length: 25.63314<br />colour: #4477AA","X: 31947.9494<br />length: 25.54396<br />colour: #4477AA","X: 32613.5316<br />length: 25.44564<br />colour: #4477AA","X: 33279.1139<br />length: 25.34253<br />colour: #4477AA","X: 33944.6962<br />length: 25.23897<br />colour: #4477AA","X: 34610.2785<br />length: 25.13932<br />colour: #4477AA","X: 35275.8608<br />length: 25.04787<br />colour: #4477AA","X: 35941.4430<br />length: 24.96663<br />colour: #4477AA","X: 36607.0253<br />length: 24.89395<br />colour: #4477AA","X: 37272.6076<br />length: 24.82793<br />colour: #4477AA","X: 37938.1899<br />length: 24.76662<br />colour: #4477AA","X: 38603.7722<br />length: 24.70810<br />colour: #4477AA","X: 39269.3544<br />length: 24.65045<br />colour: #4477AA","X: 39934.9367<br />length: 24.59173<br />colour: #4477AA","X: 40600.5190<br />length: 24.53002<br />colour: #4477AA","X: 41266.1013<br />length: 24.46350<br />colour: #4477AA","X: 41931.6835<br />length: 24.39249<br />colour: #4477AA","X: 42597.2658<br />length: 24.31916<br />colour: #4477AA","X: 43262.8481<br />length: 24.24573<br />colour: #4477AA","X: 43928.4304<br />length: 24.17441<br />colour: #4477AA","X: 44594.0127<br />length: 24.10742<br />colour: #4477AA","X: 45259.5949<br />length: 24.04699<br />colour: #4477AA","X: 45925.1772<br />length: 23.99533<br />colour: #4477AA","X: 46590.7595<br />length: 23.95468<br />colour: #4477AA","X: 47256.3418<br />length: 23.92694<br />colour: #4477AA","X: 47921.9241<br />length: 23.91169<br />colour: #4477AA","X: 48587.5063<br />length: 23.90737<br />colour: #4477AA","X: 49253.0886<br />length: 23.91242<br />colour: #4477AA","X: 49918.6709<br />length: 23.92529<br />colour: #4477AA","X: 50584.2532<br />length: 23.94440<br />colour: #4477AA","X: 51249.8354<br />length: 23.96820<br />colour: #4477AA","X: 51915.4177<br />length: 23.99512<br />colour: #4477AA","X: 52581.0000<br />length: 24.02360<br />colour: #4477AA"],"type":"scatter","mode":"lines","name":"fitted values","line":{"width":3.77952755905512,"color":"rgba(68,119,170,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[0,665.582278481013,1331.16455696203,1996.74683544304,2662.32911392405,3327.91139240506,3993.49367088608,4659.07594936709,5324.6582278481,5990.24050632911,6655.82278481013,7321.40506329114,7986.98734177215,8652.56962025316,9318.15189873418,9983.73417721519,10649.3164556962,11314.8987341772,11980.4810126582,12646.0632911392,13311.6455696203,13977.2278481013,14642.8101265823,15308.3924050633,15973.9746835443,16639.5569620253,17305.1392405063,17970.7215189873,18636.3037974684,19301.8860759494,19967.4683544304,20633.0506329114,21298.6329113924,21964.2151898734,22629.7974683544,23295.3797468354,23960.9620253165,24626.5443037975,25292.1265822785,25957.7088607595,26623.2911392405,27288.8734177215,27954.4556962025,28620.0379746835,29285.6202531646,29951.2025316456,30616.7848101266,31282.3670886076,31947.9493670886,32613.5316455696,33279.1139240506,33944.6962025316,34610.2784810127,35275.8607594937,35941.4430379747,36607.0253164557,37272.6075949367,37938.1898734177,38603.7721518987,39269.3544303797,39934.9367088608,40600.5189873418,41266.1012658228,41931.6835443038,42597.2658227848,43262.8481012658,43928.4303797468,44594.0126582278,45259.5949367089,45925.1772151899,46590.7594936709,47256.3417721519,47921.9240506329,48587.5063291139,49253.0886075949,49918.6708860759,50584.253164557,51249.835443038,51915.417721519,52581,52581,52581,51915.417721519,51249.835443038,50584.253164557,49918.6708860759,49253.0886075949,48587.5063291139,47921.9240506329,47256.3417721519,46590.7594936709,45925.1772151899,45259.5949367089,44594.0126582278,43928.4303797468,43262.8481012658,42597.2658227848,41931.6835443038,41266.1012658228,40600.5189873418,39934.9367088608,39269.3544303797,38603.7721518987,37938.1898734177,37272.6075949367,36607.0253164557,35941.4430379747,35275.8607594937,34610.2784810127,33944.6962025316,33279.1139240506,32613.5316455696,31947.9493670886,31282.3670886076,30616.7848101266,29951.2025316456,29285.6202531646,28620.0379746835,27954.4556962025,27288.8734177215,26623.2911392405,25957.7088607595,25292.1265822785,24626.5443037975,23960.9620253165,23295.3797468354,22629.7974683544,21964.2151898734,21298.6329113924,20633.0506329114,19967.4683544304,19301.8860759494,18636.3037974684,17970.7215189873,17305.1392405063,16639.5569620253,15973.9746835443,15308.3924050633,14642.8101265823,13977.2278481013,13311.6455696203,12646.0632911392,11980.4810126582,11314.8987341772,10649.3164556962,9983.73417721519,9318.15189873418,8652.56962025316,7986.98734177215,7321.40506329114,6655.82278481013,5990.24050632911,5324.6582278481,4659.07594936709,3993.49367088608,3327.91139240506,2662.32911392405,1996.74683544304,1331.16455696203,665.582278481013,0,0],"y":[24.3286475009018,24.4706503712777,24.6100360163482,24.7447274424803,24.872630963029,24.9924277533795,25.1042059329015,25.2087168387248,25.3063738286653,25.3970150830819,25.4808035816413,25.5592662207885,25.633915152876,25.7062989414171,25.7784972131713,25.8534703466669,25.9348048919624,26.0261985303771,26.130990232537,26.2475293226092,26.3684488614606,26.4858730180319,26.5920314601532,26.6797741318106,26.7428379404424,26.775506701724,26.7721091854885,26.7274883331176,26.6448954543422,26.5329048843583,26.4000207744769,26.2549508846427,26.1071241883731,25.9668328494374,25.8448031020601,25.7517263487644,25.6958836201091,25.6732567733224,25.6753699115978,25.6936375263874,25.7197900675988,25.7463638326802,25.7667009666233,25.7744595728805,25.7631947511859,25.7285854347213,25.6729433593758,25.5995531950909,25.5116647801065,25.4129620231028,25.3079920229908,25.2020230922511,25.1005271520125,25.0087660079196,24.9289285939848,24.8585985720466,24.7947543225335,24.7344191799713,24.6751633870184,24.6154441497449,24.5543389504355,24.4910307703258,24.4245796762512,24.355258539859,24.2843440449925,24.2129517853063,24.1423084856721,24.0742670423007,24.0115611029791,23.9574553787462,23.9152509919331,23.8876504156227,23.8740828442957,23.872090922144,23.8786284973822,23.8903114384802,23.904418830276,23.9196367011582,23.9354259423982,23.9512307527886,23.9512307527886,24.095966418876,24.0548059864494,24.0167537787093,23.984377788285,23.9602627002849,23.9462183369344,23.9426475769082,23.9492900825133,23.9662234958008,23.994100419345,24.0332134740303,24.0824175869741,24.1405731394159,24.206504101303,24.2785033793312,24.3539831286196,24.4297293329483,24.5024168448806,24.5690030153244,24.629123464731,24.6854578822179,24.7410459966087,24.7988220377756,24.8610999593765,24.9293067516575,25.0043224967319,25.0869823226161,25.178106887971,25.2759175209848,25.377060290739,25.4783139129532,25.5762534946988,25.6667329294777,25.7447429205611,25.8048401007797,25.841615934666,25.8514538709845,25.8395762176789,25.8144600793454,25.78469096453,25.7585384233178,25.7434661582504,25.7461320243288,25.7728779180985,25.8301475320383,25.9210577678041,26.0386324102049,26.174303922272,26.3195395987377,26.4653726638795,26.6019731517227,26.7187898827202,26.8050680686911,26.8503254997312,26.8509006039881,26.8135461197112,26.7461197687368,26.6564343187435,26.5517556291728,26.4384625960138,26.3223138391001,26.2089624793754,26.1040357000206,26.0092756849345,25.9231094290601,25.8440488052323,25.7704945550255,25.7002212484839,25.6301227038827,25.556561676251,25.4758645094572,25.3849469069909,25.2839240745148,25.1747625862322,25.0600175959144,24.9425822366469,24.8246864237551,24.7071531285798,24.5900304604316,24.4733832216332,24.3286475009018],"text":["X:     0.0000<br />length: 24.40102<br />colour: #4477AA","X:   665.5823<br />length: 24.53034<br />colour: #4477AA","X:  1331.1646<br />length: 24.65859<br />colour: #4477AA","X:  1996.7468<br />length: 24.78471<br />colour: #4477AA","X:  2662.3291<br />length: 24.90761<br />colour: #4477AA","X:  3327.9114<br />length: 25.02622<br />colour: #4477AA","X:  3993.4937<br />length: 25.13948<br />colour: #4477AA","X:  4659.0759<br />length: 25.24632<br />colour: #4477AA","X:  5324.6582<br />length: 25.34566<br />colour: #4477AA","X:  5990.2405<br />length: 25.43644<br />colour: #4477AA","X:  6655.8228<br />length: 25.51868<br />colour: #4477AA","X:  7321.4051<br />length: 25.59469<br />colour: #4477AA","X:  7986.9873<br />length: 25.66707<br />colour: #4477AA","X:  8652.5696<br />length: 25.73840<br />colour: #4477AA","X:  9318.1519<br />length: 25.81127<br />colour: #4477AA","X:  9983.7342<br />length: 25.88829<br />colour: #4477AA","X: 10649.3165<br />length: 25.97204<br />colour: #4477AA","X: 11314.8987<br />length: 26.06512<br />colour: #4477AA","X: 11980.4810<br />length: 26.16998<br />colour: #4477AA","X: 12646.0633<br />length: 26.28492<br />colour: #4477AA","X: 13311.6456<br />length: 26.40346<br />colour: #4477AA","X: 13977.2278<br />length: 26.51881<br />colour: #4477AA","X: 14642.8101<br />length: 26.62423<br />colour: #4477AA","X: 15308.3924<br />length: 26.71295<br />colour: #4477AA","X: 15973.9747<br />length: 26.77819<br />colour: #4477AA","X: 16639.5570<br />length: 26.81320<br />colour: #4477AA","X: 17305.1392<br />length: 26.81122<br />colour: #4477AA","X: 17970.7215<br />length: 26.76628<br />colour: #4477AA","X: 18636.3038<br />length: 26.68184<br />colour: #4477AA","X: 19301.8861<br />length: 26.56744<br />colour: #4477AA","X: 19967.4684<br />length: 26.43270<br />colour: #4477AA","X: 20633.0506<br />length: 26.28725<br />colour: #4477AA","X: 21298.6329<br />length: 26.14071<br />colour: #4477AA","X: 21964.2152<br />length: 26.00273<br />colour: #4477AA","X: 22629.7975<br />length: 25.88293<br />colour: #4477AA","X: 23295.3797<br />length: 25.79094<br />colour: #4477AA","X: 23960.9620<br />length: 25.73438<br />colour: #4477AA","X: 24626.5443<br />length: 25.70969<br />colour: #4477AA","X: 25292.1266<br />length: 25.70942<br />colour: #4477AA","X: 25957.7089<br />length: 25.72609<br />colour: #4477AA","X: 26623.2911<br />length: 25.75224<br />colour: #4477AA","X: 27288.8734<br />length: 25.78041<br />colour: #4477AA","X: 27954.4557<br />length: 25.80314<br />colour: #4477AA","X: 28620.0380<br />length: 25.81296<br />colour: #4477AA","X: 29285.6203<br />length: 25.80241<br />colour: #4477AA","X: 29951.2025<br />length: 25.76671<br />colour: #4477AA","X: 30616.7848<br />length: 25.70884<br />colour: #4477AA","X: 31282.3671<br />length: 25.63314<br />colour: #4477AA","X: 31947.9494<br />length: 25.54396<br />colour: #4477AA","X: 32613.5316<br />length: 25.44564<br />colour: #4477AA","X: 33279.1139<br />length: 25.34253<br />colour: #4477AA","X: 33944.6962<br />length: 25.23897<br />colour: #4477AA","X: 34610.2785<br />length: 25.13932<br />colour: #4477AA","X: 35275.8608<br />length: 25.04787<br />colour: #4477AA","X: 35941.4430<br />length: 24.96663<br />colour: #4477AA","X: 36607.0253<br />length: 24.89395<br />colour: #4477AA","X: 37272.6076<br />length: 24.82793<br />colour: #4477AA","X: 37938.1899<br />length: 24.76662<br />colour: #4477AA","X: 38603.7722<br />length: 24.70810<br />colour: #4477AA","X: 39269.3544<br />length: 24.65045<br />colour: #4477AA","X: 39934.9367<br />length: 24.59173<br />colour: #4477AA","X: 40600.5190<br />length: 24.53002<br />colour: #4477AA","X: 41266.1013<br />length: 24.46350<br />colour: #4477AA","X: 41931.6835<br />length: 24.39249<br />colour: #4477AA","X: 42597.2658<br />length: 24.31916<br />colour: #4477AA","X: 43262.8481<br />length: 24.24573<br />colour: #4477AA","X: 43928.4304<br />length: 24.17441<br />colour: #4477AA","X: 44594.0127<br />length: 24.10742<br />colour: #4477AA","X: 45259.5949<br />length: 24.04699<br />colour: #4477AA","X: 45925.1772<br />length: 23.99533<br />colour: #4477AA","X: 46590.7595<br />length: 23.95468<br />colour: #4477AA","X: 47256.3418<br />length: 23.92694<br />colour: #4477AA","X: 47921.9241<br />length: 23.91169<br />colour: #4477AA","X: 48587.5063<br />length: 23.90737<br />colour: #4477AA","X: 49253.0886<br />length: 23.91242<br />colour: #4477AA","X: 49918.6709<br />length: 23.92529<br />colour: #4477AA","X: 50584.2532<br />length: 23.94440<br />colour: #4477AA","X: 51249.8354<br />length: 23.96820<br />colour: #4477AA","X: 51915.4177<br />length: 23.99512<br />colour: #4477AA","X: 52581.0000<br />length: 24.02360<br />colour: #4477AA","X: 52581.0000<br />length: 24.02360<br />colour: #4477AA","X: 52581.0000<br />length: 24.02360<br />colour: #4477AA","X: 51915.4177<br />length: 23.99512<br />colour: #4477AA","X: 51249.8354<br />length: 23.96820<br />colour: #4477AA","X: 50584.2532<br />length: 23.94440<br />colour: #4477AA","X: 49918.6709<br />length: 23.92529<br />colour: #4477AA","X: 49253.0886<br />length: 23.91242<br />colour: #4477AA","X: 48587.5063<br />length: 23.90737<br />colour: #4477AA","X: 47921.9241<br />length: 23.91169<br />colour: #4477AA","X: 47256.3418<br />length: 23.92694<br />colour: #4477AA","X: 46590.7595<br />length: 23.95468<br />colour: #4477AA","X: 45925.1772<br />length: 23.99533<br />colour: #4477AA","X: 45259.5949<br />length: 24.04699<br />colour: #4477AA","X: 44594.0127<br />length: 24.10742<br />colour: #4477AA","X: 43928.4304<br />length: 24.17441<br />colour: #4477AA","X: 43262.8481<br />length: 24.24573<br />colour: #4477AA","X: 42597.2658<br />length: 24.31916<br />colour: #4477AA","X: 41931.6835<br />length: 24.39249<br />colour: #4477AA","X: 41266.1013<br />length: 24.46350<br />colour: #4477AA","X: 40600.5190<br />length: 24.53002<br />colour: #4477AA","X: 39934.9367<br />length: 24.59173<br />colour: #4477AA","X: 39269.3544<br />length: 24.65045<br />colour: #4477AA","X: 38603.7722<br />length: 24.70810<br />colour: #4477AA","X: 37938.1899<br />length: 24.76662<br />colour: #4477AA","X: 37272.6076<br />length: 24.82793<br />colour: #4477AA","X: 36607.0253<br />length: 24.89395<br />colour: #4477AA","X: 35941.4430<br />length: 24.96663<br />colour: #4477AA","X: 35275.8608<br />length: 25.04787<br />colour: #4477AA","X: 34610.2785<br />length: 25.13932<br />colour: #4477AA","X: 33944.6962<br />length: 25.23897<br />colour: #4477AA","X: 33279.1139<br />length: 25.34253<br />colour: #4477AA","X: 32613.5316<br />length: 25.44564<br />colour: #4477AA","X: 31947.9494<br />length: 25.54396<br />colour: #4477AA","X: 31282.3671<br />length: 25.63314<br />colour: #4477AA","X: 30616.7848<br />length: 25.70884<br />colour: #4477AA","X: 29951.2025<br />length: 25.76671<br />colour: #4477AA","X: 29285.6203<br />length: 25.80241<br />colour: #4477AA","X: 28620.0380<br />length: 25.81296<br />colour: #4477AA","X: 27954.4557<br />length: 25.80314<br />colour: #4477AA","X: 27288.8734<br />length: 25.78041<br />colour: #4477AA","X: 26623.2911<br />length: 25.75224<br />colour: #4477AA","X: 25957.7089<br />length: 25.72609<br />colour: #4477AA","X: 25292.1266<br />length: 25.70942<br />colour: #4477AA","X: 24626.5443<br />length: 25.70969<br />colour: #4477AA","X: 23960.9620<br />length: 25.73438<br />colour: #4477AA","X: 23295.3797<br />length: 25.79094<br />colour: #4477AA","X: 22629.7975<br />length: 25.88293<br />colour: #4477AA","X: 21964.2152<br />length: 26.00273<br />colour: #4477AA","X: 21298.6329<br />length: 26.14071<br />colour: #4477AA","X: 20633.0506<br />length: 26.28725<br />colour: #4477AA","X: 19967.4684<br />length: 26.43270<br />colour: #4477AA","X: 19301.8861<br />length: 26.56744<br />colour: #4477AA","X: 18636.3038<br />length: 26.68184<br />colour: #4477AA","X: 17970.7215<br />length: 26.76628<br />colour: #4477AA","X: 17305.1392<br />length: 26.81122<br />colour: #4477AA","X: 16639.5570<br />length: 26.81320<br />colour: #4477AA","X: 15973.9747<br />length: 26.77819<br />colour: #4477AA","X: 15308.3924<br />length: 26.71295<br />colour: #4477AA","X: 14642.8101<br />length: 26.62423<br />colour: #4477AA","X: 13977.2278<br />length: 26.51881<br />colour: #4477AA","X: 13311.6456<br />length: 26.40346<br />colour: #4477AA","X: 12646.0633<br />length: 26.28492<br />colour: #4477AA","X: 11980.4810<br />length: 26.16998<br />colour: #4477AA","X: 11314.8987<br />length: 26.06512<br />colour: #4477AA","X: 10649.3165<br />length: 25.97204<br />colour: #4477AA","X:  9983.7342<br />length: 25.88829<br />colour: #4477AA","X:  9318.1519<br />length: 25.81127<br />colour: #4477AA","X:  8652.5696<br />length: 25.73840<br />colour: #4477AA","X:  7986.9873<br />length: 25.66707<br />colour: #4477AA","X:  7321.4051<br />length: 25.59469<br />colour: #4477AA","X:  6655.8228<br />length: 25.51868<br />colour: #4477AA","X:  5990.2405<br />length: 25.43644<br />colour: #4477AA","X:  5324.6582<br />length: 25.34566<br />colour: #4477AA","X:  4659.0759<br />length: 25.24632<br />colour: #4477AA","X:  3993.4937<br />length: 25.13948<br />colour: #4477AA","X:  3327.9114<br />length: 25.02622<br />colour: #4477AA","X:  2662.3291<br />length: 24.90761<br />colour: #4477AA","X:  1996.7468<br />length: 24.78471<br />colour: #4477AA","X:  1331.1646<br />length: 24.65859<br />colour: #4477AA","X:   665.5823<br />length: 24.53034<br />colour: #4477AA","X:     0.0000<br />length: 24.40102<br />colour: #4477AA","X:     0.0000<br />length: 24.40102<br />colour: #4477AA"],"type":"scatter","mode":"lines","line":{"width":3.77952755905512,"color":"rgba(68,119,170,0.4)","dash":"solid"},"fill":"toself","fillcolor":"rgba(153,153,153,0.4)","hoveron":"points","hoverinfo":"x+y","showlegend":false,"xaxis":"x","yaxis":"y","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":40.1826484018265,"l":37.2602739726027},"plot_bgcolor":"rgba(235,235,235,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"title":{"text":"Animacja długości śledzia","font":{"color":"rgba(0,0,0,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-2629.05,55210.05],"tickmode":"array","ticktext":["0","10000","20000","30000","40000","50000"],"tickvals":[0,10000,20000,30000,40000,50000],"categoryorder":"array","categoryarray":["0","10000","20000","30000","40000","50000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":{"text":"Numer obserwacji","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[23.7231504380518,26.9998410880803],"tickmode":"array","ticktext":["24","25","26"],"tickvals":[24,25,26],"categoryorder":"array","categoryarray":["24","25","26"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":{"text":"Długość śledzia [cm]","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895}},"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"source":"A","attrs":{"14946fb5351f":{"x":{},"y":{},"colour":{},"type":"scatter"}},"cur_data":"14946fb5351f","visdat":{"14946fb5351f":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>
```

```r
max_length <- layer_data(p)
max_length <- max_length[(which.max(max_length$y)),1:2]


head(max_length)
```

```
##           x       y
## 26 16639.56 26.8132
```



Poniżej przedstawione histogramy przedstawiają liczbę złowionych śledzi o danej długości z podziałem na miesiące. Można zauważyć, że najwięcej zebrano w sierpniu.  Ciekawym przypadkiem jest brak połowów Śledzi o długości ok. 24cm. 

```r
ggplot(herring, aes(x=length)) + 
  geom_histogram(bins=30)+ facet_wrap(xmonth ~ .) + 
  theme(panel.grid.major = element_line(colour = "blue")) +
  scale_x_continuous(breaks = round(seq(min(herring$length), max(herring$length), by = 2),1)) +
  ggtitle("Histogramy długości śledzia dla danego miesiąca") +
  xlab("Długość śledzia [cm]") +
  ylab("Liczba śledzi")
```

![](Agata-Kaczmarek-140716_files/figure-html/hist_length-1.png)<!-- -->




Poniższy wykres przedstawia linie trendu dostępności planktonów. Nawiększe zagęszczenie mają 2. gatunek widłogonów, a najmniejszy 1. gatunek **Calanus finmarchicus**.


```r
ggplot(herring, aes(x=length)) + 
  geom_smooth(aes(y = cfin1,colour="cfin1", color = "#4477AA")) + 
  geom_smooth(aes(y = cfin2,colour="cfin2", color="#EE6677")) + 
  geom_smooth(aes(y = chel1,colour="chel1", color="#228833")) +
  geom_smooth(aes(y = chel2,colour="chel2", color="#CCBB44")) +
  geom_smooth(aes(y = lcop1,colour="lcop1", color="#66CCEE")) +
  geom_smooth(aes(y = lcop2,colour="lcop2", color="#AA3377")) +
  scale_colour_manual(name="legend", values=c("#4477AA", "#EE6677","#228833","#CCBB44","#66CCEE","#AA3377")) +
  ylab(bquote("Plankton availability")) + 
  ggtitle("Linie trendu dostępności planktonów") +
  xlab("Długość śledzia [cm]") +
  ylab("Dostępnośc planktonu")
```

![](Agata-Kaczmarek-140716_files/figure-html/plankton_geom_smooth-1.png)<!-- -->


Sprawdzono korelacje między zmiennymi. Nie sprawdzana była koorelacja numeru obserwacji z innymi atrybutami. Długość śledzia ma najsilniejszy współczynnik (-0.5) korelacji z temperaturą przy powierzchni wody (sst). Największy współczynnik korelacji (0.9) występuje dla zagęszczenia planktonów: **Calanus finmarchicus gat. 2** (chel2) i widłonogów gat. 2 (lcop2). Miesiące mają nasłabsze współczynniki korelacji z innymi atrybutami.   



```r
data(herring)
corr <- round(cor(herring[-1]),1)
ggcorrplot(corr, lab=TRUE, title="") + 
    ggtitle("Koorelacje między zmiennymi")
```

![](Agata-Kaczmarek-140716_files/figure-html/correlation-1.png)<!-- -->


Na wykresie pokazującym jak zależy długość śledzia od temperatury przy powierzchni wody można zaobserwować, że długość śledzia gwałtownie spada przy temperaturze 14 `\u00b0` C



```r
ggplot(herring) + 
  geom_smooth(aes(x=sst, y=length)) + 
  ggtitle("Trend długości śledzia zależny od temperatury") +
  xlab("Temperatura \u00b0C") +
  ylab("Długość śledzia [cm]")
```

![](Agata-Kaczmarek-140716_files/figure-html/lenght_temp-1.png)<!-- -->

Sprawdzono jak zależą od siebie atrybuty dla największego współczynnika koorelacji:


```r
ggplot(herring) + 
  geom_smooth(aes(x=lcop2, y=chel2)) + 
  ggtitle("Trend zagęszczenia widłonogów gat. 2 zależny od \nzagęszczenia Calanus helgolandicus gat. 2")
```

![](Agata-Kaczmarek-140716_files/figure-html/max_c-1.png)<!-- -->

Wysoki współczynnik koorelacji (0,7) mają atrybuty: natężenie połowów w regionie (fbar) i łączne roczne natężenie w regionie (cumf). Natomiast wysoki ujemny współczynnik (-0,7) koorelacji mają łączna liczba ryb złowiona w ramach połowu (totaln) i łączne roczne natężenie w regionie (cumf).


```r
ggplot(herring) + 
  geom_smooth(aes(x=cumf, y=fbar))+facet_wrap(vars(xmonth)) +
  ggtitle("Trend natężenia połowów w regionie") +
  xlab("Roczne natężenie połowów") +
  ylab("Natężenie połowów")
```

![](Agata-Kaczmarek-140716_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

```r
ggplot(herring) + 
  geom_smooth(aes(x=cumf, y=totaln))+facet_wrap(vars(xmonth)) +
  ggtitle("Trend łącznej liczby złowionych ryb i natężenia połowu w regionie") +
  xlab("Natężenie połowów w regionie") +
  ylab("Łączna liczba złowionych ryb")
```

![](Agata-Kaczmarek-140716_files/figure-html/unnamed-chunk-1-2.png)<!-- -->

Brak wpływu na długość śledzia mają roczny narybek (recr) oraz łączne roczne natężenie połowów w regionie (cumf).


```r
ggplot(herring) + 
  geom_smooth(aes(x=length, y=recr))+facet_wrap(vars(xmonth)) +
  ggtitle("Trend rocznego narybku i długości śledzia") +
  xlab("Długość śledzia") +
  ylab("Roczny narybek")
```

![](Agata-Kaczmarek-140716_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
ggplot(herring) +
  geom_smooth(aes(x=length, y=cumf))+facet_wrap(vars(xmonth)) +
  ggtitle("Trend łącznego rocznego natężenia połowów w regionie i długości") +
  xlab("Łączne roczne natężenie połowów w regionie") +
  ylab("Łączna liczba złowionych ryb")
```

![](Agata-Kaczmarek-140716_files/figure-html/unnamed-chunk-2-2.png)<!-- -->


# Regresor przewidujący rozmiar śledzia

## Przygotowanie danych

Dokonano podziału zbioru danych na uczące, walidujące i testowe. Zbiór uczący to 75% całego zbioru. Usunięto atrybut X - numer obserwacji. Przygotowano schemat uczenia na podstawie powwtarzającej się oceny krzyżowej (repeatedcv) z 2 podziałami i 5 powtórzeniami. 


```r
set.seed(23)
inTraining <- 
    createDataPartition(
        y = herring$length,
        p = .75,
        list = FALSE)

herringWithoutX<-select(herring, -X)

training <- herringWithoutX[ inTraining,]
testing  <- herringWithoutX[-inTraining,]

ctrl <- trainControl(
    method = "repeatedcv",
    number = 2,
    repeats = 5)
```


## Linear Regression

Przygotowano uczenie przy pomocy regresji liniowej.


```r
fit_lr <- train(length ~ .,
             data = training,
             method = "lm",
             trControl = ctrl,
             ntree = 10)
```

Dla regresji liniowej miara $R^2$ wynosi 0.32, a $RMSE$ 1.36. Najbardziej istotną zmienną jest fbar - natężenie połowóW w regionie. Pokazano podsumowanie dotyczące predykcji dla regresji liniowej.


```r
fit_lr
```

```
## Linear Regression 
## 
## 39438 samples
##    14 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (2 fold, repeated 5 times) 
## Summary of sample sizes: 19720, 19718, 19719, 19719, 19719, 19719, ... 
## Resampling results:
## 
##   RMSE      Rsquared   MAE     
##   1.363931  0.3198441  1.084964
## 
## Tuning parameter 'intercept' was held constant at a value of TRUE
```

```r
varImp(fit_lr)
```

```
## lm variable importance
## 
##          Overall
## fbar   100.00000
## cumf    89.52422
## sst     86.92289
## cfin1   24.20914
## lcop1   15.79213
## recr    15.68082
## totaln  12.80698
## nao      5.79945
## lcop2    5.61337
## chel1    4.69380
## cfin2    0.18658
## xmonth   0.14640
## sal      0.07872
## chel2    0.00000
```

```r
ggplot(varImp(fit_lr))
```

![](Agata-Kaczmarek-140716_files/figure-html/lr_plot-1.png)<!-- -->

```r
predictions_lr <- predict(fit_lr,herringWithoutX)
print(summary(predictions_lr))
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   22.75   24.72   25.44   25.30   25.87   32.33
```

## Random Forrest

Dla porównania wykorzystano również algorytmu Random Forrest.


```r
fit_rf <- train(length ~ .,
             data = training,
             method = "rf",
             trControl = ctrl,
             ntree = 10)
```


Dla Random Forrest miara $R^2$ waha się między 0.498 - 0.515, a $RMSE$ 1.148 - 1.170. Najbardziej istotną zmienną jest sst - temperatura przy powierzchni wody. Pokazano podsumowanie dotyczące predykcji dla algorytmy Random Forrest.



```r
fit_rf
```

```
## Random Forest 
## 
## 39438 samples
##    14 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (2 fold, repeated 5 times) 
## Summary of sample sizes: 19719, 19719, 19719, 19719, 19718, 19720, ... 
## Resampling results across tuning parameters:
## 
##   mtry  RMSE      Rsquared   MAE      
##    2    1.170689  0.4989424  0.9256144
##    8    1.148605  0.5177060  0.9047644
##   14    1.151361  0.5158831  0.9055804
## 
## RMSE was used to select the optimal model using the smallest value.
## The final value used for the model was mtry = 8.
```

```r
varImp(fit_rf)
```

```
## rf variable importance
## 
##         Overall
## sst    100.0000
## recr    22.1442
## xmonth  14.1005
## fbar     9.9004
## lcop2    8.5712
## cfin2    8.1121
## lcop1    7.9122
## totaln   7.3580
## nao      4.7147
## chel2    4.1836
## chel1    2.9221
## cumf     1.2219
## sal      0.6018
## cfin1    0.0000
```

```r
ggplot(varImp(fit_rf))
```

![](Agata-Kaczmarek-140716_files/figure-html/rf_plot-1.png)<!-- -->

```r
predictions_rf <- predict(fit_rf,herringWithoutX)
print(summary(predictions_rf))
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   20.82   24.60   25.34   25.31   26.23   28.41
```

## Porównanie

Miara $R^2$ jest większa dla regresji liniowej. Błąd średniokwadratowy jest mniejszy dla algorytmu Random Forrest. Najbardziej istotna zmienna dla Random Forrest sst -  temperatura przy powierzchni wody w regresji liniowej występuje z wysoką wartością na 3 miejscu. Natomiast fbar natężenie połowów w regionie z niską wartością dla Random Forrest występuje na 4 miejscu.
