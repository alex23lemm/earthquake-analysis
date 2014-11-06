Earthquake activity analysis 2014 for Darmstadt, Germany
========================================================

Synopysis
---------

In May, June, September and October 2014 several earthquakes struck the
region east of Darmstadt (Germany). In this report we will use the
earthquake data provided by the [Bundesanstalt fuer Geowissenschaften
und Rohstoffe
(BGR)](http://www.bgr.bund.de/DE/Themen/Erdbeben-Gefaehrdungsanalysen/Seismologie/Seismologie/Erdbebenauswertung/D_seit_1968/d_1968_node.html)
and the `ggmap` package to create different maps.

Loading packages
----------------

    library(ggplot2)
    library(ggmap)
    library(dplyr)
    library(httr)
    library(XML)
    library(stringr)
    library(lubridate)

Downloading and processing data
-------------------------------

We will download the data using Hadley's `httr` package and include all
earthquake records from 2014 for Germany.

Actually, the `GET` call below is not necessary. You can download the
data directly using `htmlParse` from the `XML` package. However, I just
wanted to try it out.

    page <- GET('http://www.seismologie.bgr.de/cgi-bin/eisy_test_iframe.pl?begin_date=20140101&end_date=20141106&min_lat=&max_lat=&min_lon=&max_lon=&min_mag=&max_mag=&')
    page_content <- content(page, 'text')

    parsed_content <- xmlRoot(htmlParse(page_content))

Let's take a look at the downloaded data:

    parsed_content

    ## <html xmlns="http://www.w3.org/1999/xhtml" lang="de" xml:lang="de" xml:lang="de">
    ##   <head>
    ##     <base href="http://www.seismologie.bgr.de" />
    ##     <title>BGR/Seismologie/Seismische Ereignisse in Deutschland seit 1968</title>
    ##     <meta http-equiv="content-type" content="text/html; charset=ISO-8859-1" />
    ##     <meta name="title" content="SDAC Erdbeben" />
    ##     <meta name="description" content="Seismic Data Analysis Center" />
    ##     <meta name="keywords" content="Seismologie BGR SDAC EISY Erdbeben" />
    ##     <meta name="language" content="de, deutsch, german" />
    ##     <meta name="author" content="publisher" />
    ##     <meta name="date" content="2005-12-19" />
    ##     <meta name="robots" content="index,follow" />
    ##     <link rel="stylesheet" type="text/css" href="www/gsb.css" />
    ##   </head>
    ##   <body>
    ## 
    ## 
    ## 
    ##  
    ## <h4 align="center">Liste und Karte der seismischen Ereignisse in Deutschland und benachbarten Gebieten anhand folgender Auswahlkriterien:<br /><br />Zeitraum: 20140101 - 20141106 <br /></h4><pre>DATE        TIME    LAT     LON    MAG   REGION   </pre><pre>31/10/2014  12:47   48.20   8.98   3.1   Spaichingen/Swabian Jura, S of Albstadt</pre><pre>29/10/2014  17:23   49.83   8.71   3.6   Darmstadt-East</pre><pre>10/10/2014  02:46   47.48   6.20   2.7   S of Vesoul/F</pre><pre>19/09/2014  11:11   49.94   8.69   2.6   Darmstadt-East</pre><pre>15/09/2014  13:14   49.37   6.83   2.7   Saarbruecken-West</pre><pre>11/09/2014  08:00   50.75   9.18   3.0   Alsfeld/Hessen</pre><pre>02/09/2014  00:12   52.79   8.19   2.8   Vechta, Diepholz</pre><pre>01/09/2014  07:17   53.18   6.79   3.0   Veendam/NL, SE of Groningen</pre><pre>15/08/2014  22:36   54.97  14.94   3.3   Baltic Sea, S of Bornholm</pre><pre>07/08/2014  11:31   48.05   8.10   3.0   Furtwangen/Black Forest, E of Freiburg</pre><pre>03/08/2014  23:58   50.22  12.44   3.6   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>08/06/2014  14:15   49.81   8.77   3.2   Darmstadt-East</pre><pre>02/06/2014  02:25   50.24  12.44   2.8   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>01/06/2014  00:29   50.24  12.44   2.5   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>31/05/2014  20:15   50.23  12.44   3.0   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>31/05/2014  20:14   50.23  12.44   2.7   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>31/05/2014  19:18   50.23  12.44   2.9   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>31/05/2014  13:28   50.23  12.44   3.1   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>31/05/2014  11:32   50.22  12.44   2.7   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>31/05/2014  10:43   50.23  12.44   2.9   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>31/05/2014  10:37   50.23  12.44   4.2   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>24/05/2014  14:35   50.22  12.44   3.5   Klingenthal, Kraslice/CR, Erzgebirge</pre><pre>24/05/2014  06:40   47.53  14.82   3.3   Radmer Hochkogel/A, Eisenerzer Alps</pre><pre>18/05/2014  06:41   48.20   8.98   3.0   Balingen/Swabian Jura</pre><pre>17/05/2014  16:46   49.83   8.70   3.9   Darmstadt-East</pre><pre>01/05/2014  08:29   52.89   8.76   3.2   Syke, S of Bremen</pre><pre>18/04/2014  18:12   48.27   6.58   2.8   Epinal-N/F, Rambervillers</pre><pre>30/03/2014  15:57   49.82   8.80   3.2   Darmstadt-East</pre><pre>27/03/2014  11:22   47.11  12.32   3.2   Northern Ziller Alps/A</pre><pre>06/03/2014  17:08   54.02   8.11   2.6   N of Wilhelmshaven, North Sea</pre><pre>05/03/2014  08:55   47.04   9.46   2.6   Vaduz/Liechtenstein, Swiss-Liechtenstein-Austrian border reg</pre><pre>01/03/2014  22:19   54.54   7.48   2.6   North Sea</pre><pre>13/02/2014  02:13   53.40   6.67   3.2   S. of Borkum, Ems estuary, North Sea</pre><pre>09/02/2014  15:04   48.44  14.47   2.5   Freistadt/A, NE of Linz</pre><pre>08/01/2014  18:27   47.19   7.15   3.3   Biel/CH, Bieler See</pre><pre>08/01/2014  18:12   47.19   7.13   2.9   Biel/CH, Bieler See</pre><p class="paragraph2"><span> Hinweis: Die Liste kann sowohl Erdbeben als auch Sprengungen und induzierte
    ## Ereignisse enthalten. Die Daten werden laufend Ã¼berprÃ¼ft und gegebenenfalls aktualisiert.</span></p><div class="paragraph2"><img src="http://www.seismologie.bgr.de/tmp/VFuQu8GuoyEAABN8mQAAAAD.png" alt="Deutschlandkarte mit den resultierenden Epizentren" title="Deutschlandkarte mit den resultierenden Epizentren" /><br />Topographische Deutschlandkarte mit den Epizentren, resultierend aus den eingegebenen Suchkriterien: Quelle BGR</div><p class="paragraph4"><a href="http://www.seismologie.bgr.de/sdac/erdbeben/ger_db_test_iframe.html?begin_date=20140101&amp;end_date=20141106&amp;min_lat=&amp;max_lat=&amp;min_lon=&amp;max_lon=&amp;min_mag=&amp;max_mag=&amp;" title="zurÃ¼ck zur Eingabemaske">zurÃ¼ck zur Eingabemaske</a></p></body>
    ## </html>

The relevant earthquake records are included and surrounded by `pre`
tags. We will fetch the records using a `XPath` expression and store
them in a data frame. Based on the fact that they are stored in a fixed
length format we can use a series of `str_sub` calls to extract the
relevant parts one by one.

    doc <- xpathSApply(parsed_content, '//pre', xmlValue)
    head(doc)

    ## [1] "DATE        TIME    LAT     LON    MAG   REGION   "                              
    ## [2] "31/10/2014  12:47   48.20   8.98   3.1   Spaichingen/Swabian Jura, S of Albstadt"
    ## [3] "29/10/2014  17:23   49.83   8.71   3.6   Darmstadt-East"                         
    ## [4] "10/10/2014  02:46   47.48   6.20   2.7   S of Vesoul/F"                          
    ## [5] "19/09/2014  11:11   49.94   8.69   2.6   Darmstadt-East"                         
    ## [6] "15/09/2014  13:14   49.37   6.83   2.7   Saarbruecken-West"

    data <- doc[-1]

    date <- str_sub(data, 1, 10) 
    time <- str_sub(data, 11, 17) %>% str_trim 
    lat <- str_sub(data, 18, 25) %>% str_trim 
    long <- str_sub(data, 26, 32) %>% str_trim 
    mag <- str_sub(data, 33, 38) %>% str_trim 
    region <- str_sub(data, 39, str_length(data)) %>% str_trim

    preprocessed_data <- data.frame(date, time, lat, long, mag, region, 
                                 stringsAsFactors = FALSE)

How does the pre-processed data look like?

    head(preprocessed_data)

    ##         date  time   lat long mag                                  region
    ## 1 31/10/2014 12:47 48.20 8.98 3.1 Spaichingen/Swabian Jura, S of Albstadt
    ## 2 29/10/2014 17:23 49.83 8.71 3.6                          Darmstadt-East
    ## 3 10/10/2014 02:46 47.48 6.20 2.7                           S of Vesoul/F
    ## 4 19/09/2014 11:11 49.94 8.69 2.6                          Darmstadt-East
    ## 5 15/09/2014 13:14 49.37 6.83 2.7                       Saarbruecken-West
    ## 6 11/09/2014 08:00 50.75 9.18 3.0                          Alsfeld/Hessen

Now, we are only interested in earthquakes which struck the region of
Darmstadt. In addition, we will transform some of the character
variables into numeric ones.

    # Only include earthquakes around Darmstadt and change order so that older
    # earthquakes will get plotted first in case they need to get plotted on top of 
    # each other
    processed_data <- filter(preprocessed_data, grepl('.*Darmstadt.*', region)) %>%
      transform(
        date  = dmy(date),
        lat   = as.numeric(lat),
        long  = as.numeric(long),
        mag   = as.numeric(mag)
        ) %>%
      arrange(date)

Our processed data looks as follows and can directly be used for
plotting in the next section:

    processed_data

    ##         date  time   lat long mag         region
    ## 1 2014-03-30 15:57 49.82 8.80 3.2 Darmstadt-East
    ## 2 2014-05-17 16:46 49.83 8.70 3.9 Darmstadt-East
    ## 3 2014-06-08 14:15 49.81 8.77 3.2 Darmstadt-East
    ## 4 2014-09-19 11:11 49.94 8.69 2.6 Darmstadt-East
    ## 5 2014-10-29 17:23 49.83 8.71 3.6 Darmstadt-East

Plotting data
-------------

We'll use `ggmap` and to display the center of the reported earthquakes
on a map of Darmstadt. Magnitude will be encoded by `size` and `fill`.

In our first attempt we will plot a toner map provided by
[Stamen](http://http://maps.stamen.com/#toner/).

    # Get toner map of Darmstadt from Stamen
    da_toner_map <- get_map('Darmstadt', maptype = 'toner', source = 'stamen')

    # Create toner plot object
    da_toner_plot <- ggmap(da_toner_map) +
      geom_point(aes(long, lat, size = mag, fill = mag), processed_data, shape = 21,
                 alpha = 0.6) +
      ggtitle("2014: Recent earthquakes in Darmstadt") +
      scale_size_continuous(limits = c(2,5)) +
      scale_fill_gradientn(colours = rev(terrain.colors(5)), limits=c(2,5), 
                           guide = guide_legend()) +
      labs(size = 'Magnitude', fill = 'Magnitude') +
      theme(axis.title = element_blank())

    da_toner_plot

![](./earthquake_analysis_files/figure-markdown_strict/toner_plot-1.png)

In our next plot we will use a terrain map provided by
[Google](https://maps.google.de).

    # Get terrain map of Darmstadt from Google
    da_terrain_map <- get_map('Darmstadt', zoom = 11,
                              maptype = 'terrain', source = 'google')

    # Create toner plot object
    da_terrain_plot <- ggmap(da_terrain_map) +
      geom_point(aes(long, lat, size = mag, fill = mag), processed_data, shape = 21,
                 alpha = 0.6) +
      ggtitle("2014: Recent earthquakes in Darmstadt") +
      scale_size_continuous(limits = c(2,5)) +
      scale_fill_gradientn(colours = rev(terrain.colors(5)), limits=c(2,5), 
                           guide = guide_legend()) +
      labs(size = 'Magnitude', fill = 'Magnitude') +
      theme(axis.title = element_blank())

    da_terrain_plot

![](./earthquake_analysis_files/figure-markdown_strict/terrain_plot-1.png)
