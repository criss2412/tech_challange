In primul rand am citit fisierul si am vrut sa vad cam cum arata aplicand un head 
Am dat si shape pentru a vedea cam cate randuri si coloane avem in df. 
Mai departe am vrut sa vad care este incidenta de not null-uri si pe ce coloane “ma pot baza” in unicizare, care sa fie importante si relevante. 
Din cate se poate observa, company_name si website_domain par a fi in procent de mai mult de 90% completate.

Nu as folosi in cheie status, last_updated_at,created_at pt ca nu mi se par coloane definitorii pt o cheie.

As merge pe coloane care imi pot arata unicitatea, cum ar fi company_name, website_domain si pentru ca are procent destul de mare primary_phone

Coloanele main_country_code,main_region,main_city,main_business_category,main_postcode  pot sa:

->le pastrez ca si coloane secundare si in deduplicare va iesi linia care are cele mai multe completate si le-as normaliza
->le bag in cheia principala si am granularitate mai mare pt firme cu acelasi nume, filiale diferite. 

Am facut varianta si si pentru a vedea diferentele

Coloanele emails, primary_email, le-as fi folosit in cheie dar am foarte multe valori missing

Coloanele android_app_url, ios_app_url, alexa_rank, tiktok_url, naics_2022_secondary_*, other_emails le-as elimina de tot pt ca st aproape goale toate si nu ma ajuta cu nimic


In primul rand trebuie curatate si normalizate datele, pt ca datele despre numele companiilor contin multe simboluri(virgule,puncte,cratime), stop words (inc.,co,ltd.), asa ca mai intai o sa le transform in litere mici si scoatem spatiile(trim) si dupa scoatem stop words si simbolurile. Dupa ce am aplicat functia am printat un before and after si vedem ca s-a redus nr de companii unice, ceea ce rezulta ca avem duplicate pe companie in df

Urmatoarea coloana de normalizat este nr de telefon, 
-> pentru a normaliza trebuie  sa scoatem caracterele care nu st numere
nr de telefon nu este neaparat o coloana definitorie pt a grupa, dar avem putine null-uri si ar trebui sa fie destul de unic

Urmatoarea coloana este website domain care pare destul de curat, dar tot as aplica un lower si un strip. din cate se poate vedea nu au afectat si avem 6613 de domenii diferinte..cam putine avand in vedere ca dupa normalizarea companiei avem 16302 de company name diferite, dar poate difera zona…


Pare ca si main_country_code,main_region,main_city, e ok dar am aplicat un lower ca sa fie toate la fel


Pt main_business_category avem niste caractere speciale si le-am corectat

Pt main_postcode avem aceeasi probleme cu spatiile si - si le-am corectat si pe acestea si chiar se vede o diferenta in unicitate (10221, 10284)

pt ca main_address_raw_text este un raw text si nu pare sa aiba vreo ordine anume, o sa ma folosesc de main_region+main_city+postal_code+country


Pt prima varianta de dedublare voi folosi toate coloanele precizate ca si cheie principala.
Dupa ce voi scapa de duplicate, voi utiliza fuzzy matching pentru nume de companii aproape la fel. Acesta cauta asemanari intre stringuri “aproape” la fel, pentru dublicate cu nume scris gresit sau putin diferit. Ce face algoritmul ete sa detecteze distanta dintre siruri si este o metoda buna de a scapa de duplicate scrise usor diferit.

Pentru a doua varianta de deduplicare voi folosi ca si coloane principale 'company_name_norm', 'website_domain_norm', 'primary_phone_norm' si coloane secundare  'main_city_norm', 'main_region_norm',
 'main_country_code_norm', 'main_postcode_norm','main_business_category_norm’. Asa cum am precizat si mai sus, coloanele principale le voi folosi pt a grupa datele si cele secundare in functie de cate sunt completate, linia va primi un scor si linia cea mai completa va ramane.
Similar cu prima varianta, am aplicat peste datele neduplicate si fuzzy matching pt a elimina si mai mult duplicatele aparute din cauza unor diferente minore de scriere cum ar fi Wandy’s cu Wandys de ex.

Voi lua un treshhold de 90%, deci asemanarea trebuie sa fie de 90% ca una dintre linii sa fie eliminata

Rezultatele finale sunt asa cum era de asteptat:
->pt gruparea pe mai multe coloane avem mai multe randuri 
Rânduri originale: 33446
Rânduri după deduplicare exactă: 27736
Rânduri după deduplicare fuzzy: 22411


->pentru gruparea pe mai putine coloane avem mai putine duplicate, pentru nivelul de granularitate este mai mic:

Rânduri originale: 33446
Rânduri după deduplicare exactă: 23930
Rânduri după deduplicare fuzzy: 17527

Nu am inclus in fuzzy nr de telefon pt ca exista doar partial <70%, iar gruparile vor fi mici si ratez din ele

Se poate observa ca este o diferenta semnificativa de randuri pe care algoritmul fuzzy le-a vazut ca aceeasi companie, deci concluzia este ca depinde foarte mult de ce cautam. Vrem perfect match pentru a fi unic, sau putem accepta mici diferente de scriere ale aceleiasi companii cu o rata de 90% match pentru a mai elimina din randuri.







