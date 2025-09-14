
In primul rand am citit fisierul si am vrut sa vad cam cum arata aplicand un head 

                                                        ANALIZA DATELOR
                                                        
                                                        
Primul lucru este sa vedem structura dataframe-ului cu shape.

Mai departe am vrut sa vad care este incidenta de not null-uri si pe ce coloane “ma pot baza” in unicizare, care sa fie importante si relevante. 

Din cate se poate observa, company_name si website_domain par a fi in procent de mai mult de 97% completate.

Coloanele naics_2022_secondary_codes ,naics_2022_secondary_labels, android_app_url ,ios_app_url ,alexa_rank , tiktok_url ,other_emails au un procent mai mic de 2% de completare, asa ca le voi sterge.


O companie se identifica unic printr-un cui. Aceasta poate avea mai multe sucursale in tari/orase/regiuni diferite si totusi sa fie aceeasi companie. Fara un cui, nu putem identifica exact duplicatele, dar putem considera unicitatea plecand mai intai de la company_name si website_domain, iar dupa sa consideram urmatoarele 2 cazuri:

->luam coloanele main_country_code,main_region,main_city,main_business_category,main_postcode ca si coloane secundare si pastram linia care are cele mai multe din ele completate

->mergem pe o cheie mai ampla formata din company_name si website_domain + main_country_code,main_region,main_city deoarece au o incidenta de > 88% not null si asa nu vom risca eliminarea companiilor diferite in functie de locatie


Facand o prima analiza pe date, putem vedea ca acelasi website domain apartine mai multor company_name cu nume usor diferite, cu aceeasi adresa cateodata, sau diferita. 

ex: 1300meteor.com.au

este tot timpul din AU, dar din orase diferite, cu company_name usor diferit:

1300 Meteor Used Cars
1300 Meteor Rentals Mount Isa
1300 Meteor Rentals Cairns

Pentru numele companiei vom aplica ulterior o normalizare pentru a scoate stop words, simboluri si a transforma tot numele in lower case.

La final cand vom face deduplicarea vom aplica si un algoritm fuzzy matching pentru nume de companii aproape la fel. Acesta cauta asemanari intre stringuri similare (cu un threshold ales de 90%) si detecteaza distanta dintre siruri, fiind o metoda buna de a scapa de duplicate rezultate in urma greselilor de scriere sau nume foarte similare.

Deoarece in ambele variante pentru deduplicare campurile company_name si website_domain vor fi folosite ca si cheie, voi da drop la liniile care nu au company_name si website_domain completate.


Pentru a lua ultima imagine a fiecarei companii, voi ordona companiile dupa ultimul created_at si last_updated_at considerand ca aceea ar trebui sa fie ultima imagine a fiecarei companii.

Pentru a putea totusi sa fac asta, ar trebui ca toate liniile sa aiba completate created_at si last_updated_at, dar avem 25 de randuri care nu le au completate. Am verificat si incidenta de cate nu au ambele si cate doar una, si exista doar care nu le au pe ambele. Daca existau doar cu cate una completata, puteam sa completez una in functie de cealalta, dar nu este cazul.

Pentru a nu pierde informatie, am facut o mica analiza si am unit dupa website domain numele companiei unde apare cu null pe created_dt si last_updated_date si am observat ca mai exista compania respectiva intr-un fel similar deja, deci nu este o problema sa sterg liniile cu created_at si last_updated_at null.

                                                NORMALIZAREA DATELOR

Primul camp pe care il normalizam este company_name, asa cum am precizat mai sus. Aceste contine simboluri(virgule,puncte,cratime) si stop words (inc.,co,ltd.) care trebuie eliminate. Mai intai o sa le transform in litere mici, urmand sa elimin simbolurile si stop words. Am facut si o mica analiza de before and after pentru a vedea daca am mai scapat de duplicate, si putem observa ca numarul liniilor unice a scazut -> era necesara prerocesarea.


Urmatoarea coloana de normalizat este nr de telefon, 
-> pentru a normaliza trebuie  sa scoatem caracterele care nu st numere
nr de telefon nu este neaparat o coloana definitorie pt a grupa, dar o vom folosi ca si coloana secundara

Urmatoarea coloana este website domain care pare destul de curat, dar tot as aplica un lower si un strip. din cate se poate vedea nu au afectat si avem 6613 de domenii diferinte.


Coloanele pentru adresa: main_country_code,main_region,main_city nu isi schimba count-ul dupa normalizare-> sunt curate


Pe coloana main_business_category avem niste caractere speciale pe care le-am corectat similar cu company_name. Voi folosi aceasta coloana ca si coloana secundara, deoarece procentul de valori not null este destul de redus (59%)

Pe coloana main_postcode (tot coloana secundara cu NON procent 71%) avem aceeasi probleme cu spatiile -> am corectat similar cu company_name si se vede o diferenta in unicitate (10221, 10284)


                                            DEDUPLICAREA DATELOR

Pentru prima varianta:

->luam coloanele main_country_code,main_region,main_city,main_business_category,main_postcode ca si coloane secundare si pastram linia care are cele mai multe din ele completate

Deduplicarea se va face pe coloanele principale company_name si website_domain, iar in functie de scorul primit pe coloanele secundare se va pastra una dintre linii, cea cu cele mai multe dintre coloane completate.

Ultiml pas este aplicarea algoritmului fuzzy pentru a elimina si mai multe dintre duplicate, cu nume de companie asemanator.


Rezultatele finale 

Rânduri originale: 31129
Rânduri după deduplicare exactă: 17025
Rânduri după deduplicare fuzzy: 15893


Pentru a doua varianta:

->mergem pe o cheie mai ampla formata din company_name si website_domain + main_country_code,main_region,main_city deoarece au o incidenta de > 88% not null si asa nu vom risca eliminarea companiilor diferite in functie de locatie


Deduplicarea se va face toate coloanele precizate mai sus, eliminand valorile de null si pentru main_country_code,main_region,main_city, pentru a avea unicitate, urmand sa se aplice similar algoritmul fuzzy matching pentru a elimina si mai multe dintre duplicate, cu nume de companie asemanator.


Rezultatele finale 

->pt gruparea pe mai multe coloane avem mai multe randuri pentru ca gruparea este mai ampla

Rânduri originale: 27978
Rânduri după deduplicare exactă: 18618
Rânduri după deduplicare fuzzy: 17888




Se poate observa ca este o diferenta semnificativa de randuri pe care algoritmul fuzzy le-a vazut ca aceeasi companie, deci concluzia este ca depinde foarte mult de ce cautam. Vrem perfect match pentru a fi unic, sau putem accepta mici diferente de scriere ale aceleiasi companii cu o rata de 90% match pentru a mai elimina din randuri.







