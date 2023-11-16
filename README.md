# Living in the UTF8 post latin1 world
Why you can not act as if the iron curtain still exists.

When I was very young and worked in an IBM installation, it was a struggle to have the three spacial danish characters på be displayed.
Later on - in the 80s - windows went for codepage 1252, and character based terminals now has codepages 850 and 865, and words like "Latin1" and "Western" was often heard.

Nowadays clever people has standardized UTF8 and letters from all over Europe/World can be used without problems. Or should be..

Let's see what to have an eye on, when you are dealing with system development in 2023.

Let's take a look at
- UTF8
- Sorting
- Length functions
- Upper- and lowercase
- String manipulation

We will need some data to play with. I have a list of Cities here: https://www.g12.dk/cities

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/6556f4c9-7640-4ffc-915a-98fbe1f118b8)
![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/ec7528a5-e55d-4445-b865-5b10ee57c261)

It's JSON data, and JSON is so much into UTF8. Notice the starge "\u????" codes, that are making it possible for JOSON to be used no matter what.

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
The hex value for the Polish City Łódź is c581c3b364c5ba, since UTF8 uses more than one byte for some characters. Even though, the correct length is 4. Not 7.  
I have sorted on the trimmed column "CityUp" since in PowerShell sorting does not skip spaces in front of " Belfast"  
From the abowe, I will say that PowerShell passes the test.
## Sorting
To handle sorting I zoom in on Danish, Sweedish and German letters.
I Denmark we have three extra letters ÆØÅ at after Z in the English alfabet. But we alså have a strange speciality, since double a should be considered the same as Å
That's why "Aalborg" is where it is, in the folder list from my Danish Windows PC below.

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/c8943469-58c0-461e-b196-b3720eb414de)

In Denmark and Sweden, the letter Ö is considered being next to Ø. It's like a variant of Ø  
In Gernamy - on the other hand - Ö is seen as a variation of "O". Therefor a German sorting is like this:

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/97b23935-a5df-472b-b38e-ab6fa13ce3e9)

This is from PostgreSQL. In Postgres the sorting and UTF settings is on a database level.  
In Postgres one does *not* have to trimm for " Belgrade". Postgres skippes prefixed blanks and sort correctly.  
It's a German sorting so between "Odense" and "Piteå" the cities "Öhringen", "Örebro" and "Østerlars" is found.

## PostGreSQL
When using PostgreSQL from a CMD window in Windows some tricks has to been thrown.
WIthout doing anything, one cannot just extract UTF8
``` CMD
de_tables=> select * from cities;
ERROR:  character with byte sequence 0xc5 0x81 in encoding "UTF8" has no equivalent in encoding "WIN1252"
```
Let's try to tell the Command Line tool this: `SET CLIENT_ENCODING TO 'utf8';`
Now it lokks like this;

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/236074f7-d9b9-4b15-9cfb-669623186946)

Not exactly what we were aiming at.  
You have to change into CodePage 65001 to be able to use UFT8.  
`CHCP 65001` is the command you have to use *before* you start PSQL

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/347dee85-efa1-44f3-ab17-75bb21051d04)

# Using SAS 

data cities;
  city = 'Łódź';
  len = length(city);
run;  
SAS thinks that the length is 7 insteadt of 4.  
SAS han new function for UTF8. For example klength() which return 4 for 'Łódź', and thats fine.  

Try to imagine a big organisation going from SAS(Latin) to SAS(UTF8) ?  Millions of SAS Datasets has to be convertet to UTF8 and SAS Programs will fail big time.  
The amount of problems and used hours for a big old company is hard to justify - just for being UTF8 complient. (Which - IMHO - would be the only right thing to do)  

SAS has to come up with some solution that "over night" can convert a company away from Latin into the age of UTF8.

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/84790cab-c7a0-4d0f-bafc-5f0fc54941e4)


![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/e4cdf500-c0e8-449e-9447-cd24b5d86e3e)

![image](https://github.com/ThorkilG12/Living-in-the-UTF8-post-latin1-world/assets/12120277/2cdc419c-0dde-42d6-a4ce-0d0982ffddb9)

