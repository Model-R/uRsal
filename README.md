# uRsal
repo for develpment of package to retrieve, clean and validate herbarium occurrence data Using R for South American Localities

## SCOPE

Goals: provide tools to perform geographical and taxonomic validation of herbarium records

Groups of organisms: plants, ferns and mosses??? 

Geographical delimitation: Latin America, with current focus on Brazil???

Potential users: taxonomists, collection curators, ecologists and conservationists

Notas:
O pacote será delimitado mais pelo formato de entrada dos dados que ao tipo de coleção. Mas até então, tudo foi testado apenas para dados de herbário. Mas o speciesLink tem outros tipos de coleção tb…. Ou seja, pensar se precisa ser apenas “herbarium records”. O GBIF mesmo possui muitos outros tipos de registros, que potencialmente já estariam no formato que precisamos, apesar de algumas colunas estarem potencialmente faltando...

	
## INPUT, OUTPUT AND ACCESSORY FILES

Search input: vector of species name(s), genus or families (default = NULL: no name filtering) and vector of collection acronym (default = NULL: all collections available)

Data output: cbind(input data table, new columns related to the string editing and to geographical, taxonomical and duplicate-search information and validation/flagging)

Accessory files: Gazeetter (download from a given website or github)


## ASSUMPTION OF THE DATA VALIDATION PROCESS

In case of invalid or missing coordinates, we assume that the county, state, county (and locality) are correct (i.e. locality prevails over coordinates), and so the working coordinates are taken from a gazetteer 

We ignore record coordinates given only at county level, assuming that our gazetteer is a more complete/safe source of county coordinates (this may not be the case for outside Latin America). It is also important to note that if the occurrence information on the localities are indeed mistaken (eg. wrong/missing county name), than the locality won`t be found in the gazetteer and thus, even if the original coordinates are good, they will be replaced by coordinates


## STEP BY STEP - DATA CLEANING

0. Data download (or data entry?)

  0.1 List of collections available for download
	
  0.2 Obtaining the links for the DwC, EML and RTF files and other collection metadata
	
  0.3 Downloading the content for each collection
	
    0.3.1 speciesLink
		
    0.3.2 Jabot
		
    0.3.3 GBIF
		
  0.4. Filtering of the downloaded data? By names or other fields...  

Notas: 

- Deixar esse passo para o fim: vale a pena investir nisso ou o usuário irá fornecer a lista de entrada para verificação?

- Se quisermos incluir esse passo, será necessário a consulta com speciesLink e Jabot sobre a possibilidade de consulta direta aos servidores deles para download das coleções disponíveis

- Marinez perguntar para Sidnei: podemos usar em um pacote de download de dados os links das coleções fornecidos ao Renato Lima (http://ipt1.cria.org.br/ipt/)  

- Marinez vai falar com Luis Alexandre sobre como usar o ipt do JABOT tb

1.  Data editing - names and collector number

1.1 Editing collectors and determiners names
	
  1.1.1 Getting the first author/determiners for multiple authors
		
    1.1.2 Removing unwanted characters and expressions (e.g. “et al.”)
		
    1.1.3 Resolve encoding problems
		
    1.1.4 Editing compound names (e.g. “Leitão-Filho”)

    1.1.5 Editing/removing prefixes or prepositions (e.g. de, do, dos, da…)

    1.1.6 Assigning occurrences with no collector information

    1.1.7 Getting the names in the standardized TDWG format
Functions ```format.name``` and ```format.name```

    1.1.8 Getting authors/determiners last name
Function ```last.name``` adapted from Hans ter Steege function

	1.2 Editing collectors numbers
	
		1.2.1 Removing unwanted characters, letters and expressions


Notas: 

- Atualmente Renato está fazendo a rotina de edição separada para os dados de cada rede (speciesLink, JABOT, GBIF). Pensar em maneiras de padronizar os campos e nomes dos cabeçalhos de cada rede para ter uma função única.

2. Data editing - locality info and geographical coordinates

	2.1 Editing and standardization of locality names (country, state and county)
	
    2.1.1 correct common typos
	
    2.1.2 removing unwanted characters and solving encoding problems
	
    2.1.3 Creating the locality strings: Country_State_County_Locality_Sublocality

    2.1.4 Remove prefixes or prepositions (e.g. de, do, dos, da…) from the strings

  2.2 Get coordinates from the gazetteer (county level)
  
  2.3 Get coordinates from the gazetteer (locality level)

  2.4 Downgrading the resolution for the localities not found in the gazetteer

  2.5 Editing coordinates and defining the working coordinates
	    
	    2.5.1 Plain zero coordinates as missing coordinates 
	
	    2.5.2 One of the coordinates missing as missing coordinates
	
	    2.5.3	Possible problems with decimal division (e.g. commas instead of points) 
	    
	    2.5.4 Any coordinate on non-decimal degrees format?	
	
	    2.5.5	Create the working coordinates and define the origin (original vs. gazetteer) and resolution the information (no minutes, no seconds) 
	    
	    2.5.6	Detecting probable rounding issues	
	    
	    2.5.7	Removing problematic coordinates still in the mix (e.g. lat>abs(90))

  
  2.6 Replacing missing coordinates by the county coordinates from the gazetteer 


Notas:

- Gazetteer precisa ser revisto quanto a nomenclatura das divisões administrativas entre países (e.g. Peru has region, province (estado?), distritos (municípios?)) 

- Como a inclusão no gazetteer de localidades e sub-localidades é uma novidade, ainda não há uma rotina de padronização dos nomes de localidade (e.g. Parque Nacional => PARNA ou vice-versa) 

- Há várias alterações e correções que são feitas quase manualmente. Pensar em criar um dicionário para as principais correções dos campos localidade (e.g. “S. José” =>  “São José”

- A edição dos dados de localidade do GBIF inclui uma padronização dos nomes dos estados (e.g. RJ => rio de janeiro)


3. Geographical validation

(Incluir aqui os passos que o Diogo já destrinchou)

4. Taxonomic validation

  4.1 Standardize the nomenclature (synonyms, typos) to get only valid names for all occurrences

  4.2 Consider all names at infraspecific level at specific level (e.g. remove ‘var.’, ‘subsp.’, ‘forma’, etc)

  4.3 Edit family names (i.e. making sure that family in the record and in the taxonomists dictionary is the same). Step done using ‘flora’ package.

  4.4 Cross the unique string of family-specialist combinations from the taxonomist dictionary and the family-determiner from the record

  4.5 Validating all type specimens (isotype, paratypes, holotypes, etc)
 
Notas:

- Hoje, a atribuição do mesmo nome válido para todos os nomes encontrados nos herbários é feito pelo meu dicionário de nomes. Mas para o pacote essa atribuição tem que ser automatizada (pacote ‘flora’ ou ‘taxize’?) ou baseada em uma lista fornecida pelo usuário.

- Por uma decisão minha, faço a checagem taxonômica (e as análises posteriores apenas ao nível específico), mas acho que esse passo é opcional e talvez desnecessário em um contexto de um pacote mais abrangente

5. Duplicate search

5.1 Create the tree strings to be used in the duplicate search (e.g. Family_Author_collectionNumber_County)

  5.2 Flagging the existence of duplicates (top down and bot down)
  
  5.3 Merge best-resolution info from each duplicate?

Notas:
- A fusão das informações das duplicatas é opcional mas creio que ela aportará muitas ocorrências taxonomicamente validadas para a análise
Renato ainda vai finalizar esse código... 


## SUPPORTING FILES

```gazetteer.csv```

good to use at county level (or best resolution available from GDAM 3.6 - state, country) for all Latin American countries
Renato completed to add localities from TreeCo, CNCFlora and IBGE. Done!
Renato obtained >1 million locality names from geonames for all Latin American countries, but this need checking before entering the gazetteer
Need to add more possible orthographic variants for counties and localities
Need to build a database of federal, state and municipal UCS and extract/add localities from UCs centroids for each county in the gazetteer
Need to extract to a different file the info of the localities (e.g. biome) and store it separately (e.g. gazetteer_metada.csv)
Need to check problems with county-disagreement between IBGE and GDAM - Done by Renato!
Need to define:

1. How to deal with multiple locality entries with different coordinates (within and between coordinate sources)? Return average coordinates or remove duplicates?
 
Contact people from other countries to have, in the mid term, localities for other countries of Latin America  

```autores.csv```

ready to use
need to cross-check with Flora do Brasil family specialists
Simplify the number of fields, store authors/taxonomist info separately (e.g. autores_metadata.csv) 
Include a more international ID for each name taxonomist (e.g. ORCID)

```collections.csv```
non-existent, but Renato have a list of many herbaria available in speciesLink and JABOT that can be used as a start
list of collections available for download

```species.csv``` ?? 

list of synonyms and basionyms per valid name??
Table with county-specific threshold for the maximum distance between the original coordinate and the centroid of the county?
Shape file with edited fields (fields and format matching the gazetteer) for the validation of geographical coordinates? Many discordances between county names from GDAM and IBGE, with some errors as well...
