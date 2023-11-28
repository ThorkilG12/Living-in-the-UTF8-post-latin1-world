# Living in the UTF8 post latin1 world.
Why you can not act as if the iron curtain still exists.

When I was very young and worked in an IBM installation, it was a struggle to have the three special danish characters to be displayed at all.  
Later on - in the 80s - windows went for codepage 1252, and character based terminals now has codepages 850 and 865, and words like "Latin1" and "Western" was often heard.

Nowadays clever people has standardized UTF8, and letters from all over Europe/World can be used without problems. Or should be..

Let's take a look at some issues, when you are dealing with system development in 2023.

Let's take a look at
- UTF8
- Sorting
- Length functions
- Upper- and lowercase
- String manipulation

We will need some data to play with. I have a list of Cities here: https://www.g12.dk/cities

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/6556f4c9-7640-4ffc-915a-98fbe1f118b8)
![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/ec7528a5-e55d-4445-b865-5b10ee57c261)

It's JSON data, and JSON is highly related to UTF8. Notice the strange "\u????" codes, that are making it possible for JSON to be used - no matter what.

Let's take the data into a Danish Windows 11 using PowerShell

It looks like this:

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/58ef45fc-1299-4a8c-93db-4925e2ee2590)

``` PowerShell
$cities = Invoke-RestMethod -Uri "https://www.g12.dk/cities"
$citiesPlus = @();
foreach($city in $cities) {
    $citiesPlus += [pscustomobject]@{
    "City" = $city.city; 
    "CityUp" = ($city.city.ToUpper()).Trim(); 
    "CityLow" = $city.city.ToLower(); 
    "CityLen" = $city.city.Length; 
    "CitySub" = $city.city.Substring(0,2); 
    "Text" = $city.text}
}
$citiesPlus | Sort-Object -Property CityUp | Select CityLen, CityLow, CityUp, CitySub
Get-Culture # da-DK is active
$citiesPlus | Sort-Object -Property CityUp | Out-GridView
``` 
The hex value for the Polish City "Łódź" is "c581c3b364c5ba". UTF8 uses more than one byte for some characters. Even though, the correct length is 4. Not 7.  
I have sorted on the trimmed column "CityUp" since - in PowerShell - sorting does not skip spaces in front of " Belfast"  
PowerShell has a Get- and Set-Culture. You have to restart PowerShell when you have changed Culture, but all sorting and stuff works fine.  
From the above, I will say that PowerShell passes the test.
## Sorting
To handle sorting I zoom in on Danish, Sweedish and German letters.
I Denmark we have three extra letters ÆØÅ showing up after ...w,x,y,z in the English alfabet. But we also have a strange speciality; a double "aa" should be considered the same as "Å"
That's why "Aalborg" is where it is, in the folder list from my Danish Windows PC below.

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/c8943469-58c0-461e-b196-b3720eb414de)

In Denmark and Sweden, the letter "Ö" is considered being next to "Ø". It's like a variant of "Ø"  
In Gernamy - on the other hand - "Ö" is seen as a variation of "O". Therefor a German sorting is like this:

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/97b23935-a5df-472b-b38e-ab6fa13ce3e9)

This is from PostgreSQL. In Postgres the sorting and UTF settings is set at the database level.  
In Postgres one does *not* have to trim for " Belfast". Postgres skips prefixed blanks and sort correctly.  
It's a German sorting, and therefore, between "Odense" and "Piteå" the cities "Öhringen", "Örebro" and "Østerlars" is found.

## PostGreSQL
When using PostgreSQL from a CMD window in Windows some tricks has to been thrown.
WIthout doing anything, one cannot just extract UTF8
``` CMD
de_tables=> select * from cities;
ERROR:  character with byte sequence 0xc5 0x81 in encoding "UTF8" has no equivalent in encoding "WIN1252"
```
Let's try to tell the Command Line tool this: `SET CLIENT_ENCODING TO 'utf8';`
Now it looks like this;

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/236074f7-d9b9-4b15-9cfb-669623186946)

Not exactly what we were aiming at.  
You have to change into CodePage 65001 to be able to use UFT8.  
`CHCP 65001` is the command you have to use *before* you start PSQL

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/347dee85-efa1-44f3-ab17-75bb21051d04)

From my work with PostgreSQL the last 5 years or so, I will say that Postgres pass the test witout any comments 👍

## Microsoft SQL
I was not able to figure out how to make SQL Mgmt Studio to read directly from the web. I had to Download JSON data as a file.  
Notice that `varchar` does *not* support the greek letters. I had to use `nvarchar`
``` SQL
-- https://learn.microsoft.com/en-us/sql/relational-databases/collations/collation-and-unicode-support?view=sql-server-ver16
SELECT tbl.*
into dbo.t_cities
FROM OPENROWSET (BULK 'C:\www.g12.dk.json', SINGLE_CLOB) js
CROSS APPLY OPENJSON(BulkColumn)
WITH (
    [City] [nvarchar](13),
    [Text] [nvarchar](50)
) tbl;
create view dbo.cities as
 select City
	  , trim(upper(City)) as CityUp
	  , lower(City) as CityLow
	  , len(City) as LenCity
	  , substring(City,1,2) as CitySub
	  , Text
 from [dbo].[t_cities]
;
select * from [dbo].[cities] order by CityUp COLLATE Danish_Norwegian_CI_AS
select * from [dbo].[cities] order by CityUp COLLATE Latin1_General_CI_AS
select * from [dbo].[cities] order by CityUp COLLATE Finnish_Swedish_CI_AS
```
![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/ecbeb732-f60b-4a4d-b72a-9346a11a3ccb)

Microsoft passes with bravour !
 
## Using JavaScript, PHP, Apache, Browsers and smartphones
From my point of view, the whole web and the most popular underlaying tools handles UTF8 fine.  
The transision is over 😊

## Using BOM (Byte Order Mark)
A bit out of scope for this document I have to mention BOM. It's part of the story about UTF8, and it's always annoying when it turns up.  
You can read about it in Wiki: https://en.wikipedia.org/wiki/Byte_order_mark  

Here is a little bit:
> Microsoft compilers[11] and interpreters, and many pieces of software on Microsoft Windows such as Notepad (prior to Windows 10 Build 1903[12]) treat the BOM as a required magic number rather than use heuristics. These tools add a BOM when saving text as UTF-8, and cannot interpret UTF-8 unless the BOM is present or the file contains only ASCII. Windows PowerShell (up to 5.1) will add a BOM when it saves UTF-8 XML documents. However, PowerShell Core 6 has added a -Encoding switch on some cmdlets called utf8NoBOM so that document can be saved without BOM. Google Docs also adds a BOM when converting a document to a plain text file for download. 

Your friend when having issues reading and writing files is Notepad++  
![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/8814aff2-e8c4-45ad-9d1e-c67e3bbdb9f0)


## Using SAS 
``` sas
data cities;
  city = 'Łódź';
  len = length(city);
run;
```
SAS thinks that the length is 7 insteadt of 4.  
SAS has new function for UTF8. For example `klength()` which return 4 for "Łódź", and thats fine.  

Try to imagine a big organisation going from SAS(Latin) to SAS(UTF8) ?  
Millions of SAS Datasets has to be convertet to UTF8 and SAS Programs will fail big time.  
The amount of problems and time spent for a big old company to switch to UTF is hard to justify - just for being UTF8 complient. (Which - IMHO - would be the only right thing to do)  

SAS has to come up with some solution that "over night" can convert a company away from Latin into the age of UTF8.  
If you try to use UTF8 characters within SAS(Latin) you will see this:  
![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/84790cab-c7a0-4d0f-bafc-5f0fc54941e4)

SAS Has a solution. A standars old-fashion installed PC SAS has this to offer:  
![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/e4cdf500-c0e8-449e-9447-cd24b5d86e3e)

Now you can work with UTF in SAS:  
![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/2cdc419c-0dde-42d6-a4ce-0d0982ffddb9)

When SAS starts up in UTF it has a few issues on the log, but the data gets in. SAS Dataset encoding is UTF-8
![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/223d34b4-362f-4587-aba5-13f2cf632b4d)

``` sas
data cities / view = cities;
  set t_cities(drop=text);
  CityUp = trim(upcase(city)); 
  CityLow = lowcase(city); 
  CityLen = klength(city); 
  CitySub = ksubstr(city,1,2); 
run;
proc sort danish data=cities out=cities_da;
  by CityUp;
run;
proc sort swedish data=cities out=cities_se;
  by CityUp;
run;
proc sort SORTSEQ=LINGUISTIC(COLLATION=PHONEBOOK) data=cities out=cities_de;
  by CityUp;
run;
```
It does *not* handle Danish correct.

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/bd1b3a32-c7d9-4760-9cc3-8d2ac317b411)

``` sas
options locale=da_DK;
proc sort SORTSEQ=LINGUISTIC data=cities out=cities_da_l;
  by CityUp;
run;
options locale=se_SE;
proc sort SORTSEQ=LINGUISTIC data=cities out=cities_se_l;
  by CityUp;
run;
options locale=de_DE;
proc sort SORTSEQ=LINGUISTIC data=cities out=cities_de_l;
  by CityUp;
run;
```
![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/fe1c0e22-e9e0-407e-8628-0c2adc8a1210)

So... SAS do sort accepable. A bit clumsy, but it is possible.

## Using SLC / Altair
Hi Thorkil,
Thanks for clarifying that, my misunderstanding. It seems what you really need is a SORTSEQ=LINGUISTIC on PROC SORT to handle these types of issue.
Unfortunately SLC does not support this option at present and hence why the interpretation is different.
Regards Altair
