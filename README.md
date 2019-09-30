# pmAPI
Internetový portál ČSOB POSMerchant je určen pro klienty ČSOB, kteří využívají její služby. Zejména v oblasti platebních terminálů a platební brány. Portál má dvě základní funkce. Poskytnout klientovi náhled na transakce a umožnit změnu jejich stavu. Změnou stavu se rozumí reverzovat, částečně reverzovat, zrušit reverzal. Popřípadě si o tyto operace požádat, pokud daný typ operace nad konkrétní transakcí podléhá schvalování.

Doplňkovou funkcí portálu je i zpřístupnění klientských reportů. Typ reportu, četnost a formát je individuální.

pmAPI zpřístupňuje tyto klíčové funkce klientům prostřednictvím rest služeb. Komunikace mezi informačním systémem klienta a portálem ČSOB probíhá přes šifrovaný protokol HTTPS a dále je zabezpečena pomocí elektronických podpisů všech požadavků a odpovědí. Elektronickými podpisy je navíc zaručena konzistence dat a nemožnost jejich změny třetí osobou (man-in-the-middle attack).

Tato dokumentace obsahuje popis služeb pmAPI a způsob jeho integrace na straně klienta. Součástí integrace je i zprovoznění zabezpečené komunikace, tedy vygenerování certifikátů a jejich výměna s bankou.
## 1. Slovníček nejdůležitějších pojmů
**Account Transport** - zúčtovací modul pro odbavení karet ve veřejné dopravě

**API** - rozhraní mezi Internetovým portálem ČSOB POSMerchant a informačním systémem klienta ČSOB

**Banka** - viz ČSOB

**Bezhotovostní platba** - forma platebního styku. Jde o takové platby, které probíhají prostřednictvím převodů peněžních prostředků bez fyzické potřeby peněz mezi klienty na jejich účtech u bank

**Clearingová transakce** - suma za jeden a více tapů během dne

**ČSOB** - Československá obchodní banka, a.s.

**DCC** - Dynamic Currency Conversion – služba dynamického převodu měn

**HTTP** - Hypertext Transfer Protocol - je internetový protokol určený pro výměnu dat.

**HTTPS** - Hypertext Transfer Protocol Secure - je v nadstavba síťového protokolu HTTP, která umožňuje zabezpečit spojení mezi klientem a serverem před odposloucháváním, podvržením dat a umožňuje též ověřit identitu protistrany. Přenášená data jsou šifrována pomocí SSL nebo TLS

**Klient** - klient ČSOB, využívající služeb v oblasti platebních terminálů

**MerchantID** - identifikátor obchodníka neboli „FirmaID“, které je přiděleno obchodníkovi ze strany ACQ/BANIT. ID Firmy lze dohledat v PM pod záložkou „Obchodníci“ v sekci „Výpisy“ (v případě, že jsou aktivovány) nebo je možné jej zobrazit v sekci „Obchodníci“ po kliknutí na ikonu pro generování klíčů - MerchantID (FirmaID).

**Obchodník** - viz Klient

**Plátce** - zákazník platící platební kartou na platebním terminále

**Platební karta** - nástroj určený k bezhotovostním platbám

**Platební terminál** - nebo také POS terminál (z anglického Point-of-sale). Elektronické zařízení, které umožňuje provedení bezhotovostní transakce platební kartou

**PM** – zkratka pro systém POSMerchant

**POSMerchant** - systém banky pro správu transakcí obchodníka

**Report** - data generovaná na žádost klienta. Data jsou v elektronické podobě. Nejčastěji ve formátech zip, txt, pdf, xls

**REST** - Representational state transfer - je cesta jak jednoduše vytvořit, číst, editovat nebo smazat informace ze serveru pomocí jednoduchých HTTP volání. 

**Tap** - odbavení platební karty ve vozidle veřejné dopravy

**Transakce** - záznam vzniklý při platbě na platebním terminálu. Originální transakce jsou drženy v bezpečné zóně na serverech ČSOB. POSMerchant a pmAPI má z bezpečnostních důvodů přístup pouze k obrazu transakcí, vytvářeného v pravidelných časových intervalech. Obraz transakce obsahuje pouze bezpečná data, na jejichž základě nelze transakci znovu provést

**Výpis** - viz Report
## 2. Funkce pmAPI
###     2.1. Transakce
####           2.1.1. Stav transakce
Transakce při procházení svým životním cyklem mohou nabývat mnoha různých stavů. Po vytvoření může být transakce úspěšně autorizována, nebo zamítnuta. Zamítnutí může zapříčinit mnoho různých důvodů. Např. nedostupnost autorizačního serveru, nedostatek prostředků na kartě plátce, exspirovaná platební karta… Úspěšně autorizované transakce poté putují do zúčtování. Autorizované nezaúčtované i zaúčtované transakce mohou být reverzovány a opětovně vráceny ke zpracování.

Transakce může nabývat stavů:
-	processing - ve zpracování
-	deny - zamítnuto
-	authorized - autorizováno/nezaúčtování
-	processed - autorizováno/zaúčtováno
-	reversed - reverzováno

**Stavy transakce**

![Stavy transakce](https://github.com/gbily/Test/blob/e8ab669caa1d226879c1924e5ec995ef9d9515a0/Stavy%20transakce.png "Stavy transakce")

#####           2.1.1.1. Zjištění stavu transakce payment/state
Metoda sloužící ke zjištění aktuálního stavu transakce. Obchodník si její pomocí může ověřit existenci konkrétní transakce, popřípadě zkontrolovat provedení změny stavu, zaúčtování.
#####           2.1.1.2. Změna stavu transakce
Obchodník může měnit stav transakce.

Dovolené změny stavu jsou:
-	reverzal request - zrušení transakce
-	process request - znovuproplacení zrušené transakce

Změna stavu transakce podléhá procesu schvalování ze strany banky. Obchodník tak přímo neprovádí aktivní operaci nad transakcí, ale pouze podává žádost o její provedení.

Žádost muže být:
-	open - nová žádost - nová žádost o změnu stavu transakce
-	done - provedena - daná operace je provedena, stav transakce je změněn
-	postponed - odložena - operace nad danou transakcí je prověřována, o jejím provedení bude rozhodnuto později
-	declined - zrušena - operace je zamítnuta, stav transakce není změněn

Doba vyřízení žádosti závisí na rychlosti vyřízení žádosti v bance. Žádost může být vyřízena, nebo zamítnuta.

**Stavy požadavku na změnu stavu transakce**

![Stavy požadavku na změnu stavu transakce](https://github.com/gbily/Test/blob/e8ab669caa1d226879c1924e5ec995ef9d9515a0/Stavy%20po%C5%BEadavku%20na%20zm%C4%9Bnu%20stavu%20transakce.png "Stavy požadavku na změnu stavu transakce")

U transakcí ve stavu autorizováno/zaúčtováno je možné zadat částku reverzalu/znovuproplacení. Takzvaný částečný reverzal/částečné znovuproplacení. Částka musí být větší než nula a menší nebo rovna částce transakce.

**Životní cyklus transakce**

![Životní cyklus transakce](https://github.com/gbily/Test/blob/e8ab669caa1d226879c1924e5ec995ef9d9515a0/%C5%BDivotn%C3%AD%20cyklus%20transakce.png "Životní cyklus transakce")

#####           2.1.1.3.	Reverzování transakce payment/reverse
Metoda požádá o odvolání úspěšně autorizované nebo zaúčtované transakce.

Transakce ve stavu autorizováno/nezaúčtováno bude vyřazena ze zpracování a prostředky na platební kartě plátce se uvolní, nebo nebudou strženy.

U transakce ve stavu autorizováno/nezaúčtováno lze provádět:
-	plný reverzal - reverzal na celou částku transakce - částka není zadána

Transakce ve stavu autorizováno/zaúčtováno zůstane beze změny. Ale vznikne nová transakce typu reverzal, která po projití standardního životního cyklu transakce vrátí prostředky na platební kartu plátce. 

U transakce ve stavu autorizováno/zaúčtováno lze provádět:
-	plný reverzal - reverzal na celou částku transakce - částka není zadána
-	částečný reverzal - reverzal na částku větší než nula a menší nebo rovnu částce transakce - částka je zadána
#####           2.1.1.4.	Znovuproplacení transakce payment/process
Metoda požádá o znovuproplacení reverzované transakce.

Transakce ve stavu reverzovaná, která byla reverzována ze stavu autorizováno/nezaúčtováno bude znovu zařazena ke zpracování. Její životní cyklus bude pokračovat dále a po jejím zaúčtování budou strženy prostředky z platební karty plátce.

U transakce ve stavu reverzovaná, která byla reverzována ze stavu autorizováno/nezaúčtováno lze provádět:
-	plné znovuproplacení - znovuproplacení na celou částku transakce - částka není zadána

Transakce ve stavu reverzovaná, která byla reverzována ze stavu autorizováno/zaúčtováno zůstane beze změny. Ale vznikne nová transakce, která po projití standardního životního cyklu transakce strhne prostředky z platební karty plátce.

U transakce ve stavu reverzovaná, která byla reverzována ze stavu autorizováno/zaúčtováno lze provádět:
-	plné znovuproplacení - znovuproplacení na celou částku transakce - částka není zadána
-	částečné znovuproplacení - znovuproplacení na částku větší než nula a menší nebo rovnu částce transakce - částka je zadána
###     2.2. Výpisy
####           2.2.1. Dotazy na generované výpisy
pmAPI umožňuje obchodníkovi automatizovat stahování výpisů z POSMerchantu. Pro tuto operaci potřebuje znát, jaké výpisy mu byly vygenerovány, a následně mít možnost je stáhnout.

Výpisy nejsou shodné u všech obchodníků a ne každý obchodník využívá možnosti generování výpisů. 

Výpisy mohou být generovány v časových intervalech:
-	denní
-	týdenní
-	měsíční
-	speciální

Výpisy mohou být uloženy ve formátech:
-	zip - souborový formát pro kompresi a archivaci dat
-	txt - souborový formát pro ukládání textových informací
-	pdf - Portable Document Format – Přenosný formát dokumentů je souborový formát vyvinutý firmou Adobe pro ukládání dokumentů nezávisle na softwaru i hardwaru, na kterém byly pořízeny.
-	xls - souborový formát programu Microsoft Excel vyvinutý firmou Microsoft
#####           2.2.1.1.	Seznam vygenerovaných výpisů report/list
Metoda vrátí seznam názvů souborů všech výpisů, vygenerovaných od daného data. Datum může být maximálně 1 měsíc zpět. Partnerský systém si musí pamatovat datum poslední kontroly výpisů a toto datum zaslat v následující kontrole. Pokud datum nebude uvedeno, vrátí se seznam výpisů za poslední měsíc.
#####           2.2.1.2.	Stažení konkrétního výpisu report/get
Metoda vrátí soubor požadovaného výpisu. Výpisy je možné stahovat pouze jednotlivě. Výpisy jsou nezávisle na uloženém formátu poskytovány vždy v komprimované podobě - zip.
###     2.3. Account Transport
####           2.3.1. Dotazy na zúčtovací modul
Account transport je zúčtovací modul pro odbavení bankovních karet ve veřejné dopravě. Poskytuje funkce pro agregaci jízdného, zúčtování a správu blacklistu. Account modul se stará především o agregaci plateb za jednotlivé jízdné jednou platební kartou do celkové sumy. Každé použití platební karty ve vozidle se nazývá tap. Sloučení plateb do jedné celkové sumy se nazývá vyúčtovací transakce neboli clearingová transakce. pmAPI umožňuje obchodníkovi číst seznamy tapů a clearingových transakcí ze zúčtovacího modulu.
#####           2.3.1.1.	Seznam tapů transport/tap/list
Metoda vrátí seznam jednotlivých tapů za zvolené období. Datum může být maximálně 1 měsíc zpět. Tapy lze filtrovat podle různých kritérií, která jsou popsána v ukázkách níže.
#####           2.3.1.2.	Seznam clearingových transakcí transport/clear/list
Metoda vrátí seznam jednotlivých clearingových transakcí za zvolené období. Datum může být maximálně 1 měsíc zpět. Transakce lze filtrovat podle různých kritérií, která jsou popsána v ukázkách níže.
###     2.4. DCC
####           2.4.1.	Kurzovní lístek
pmAPI umožňuje obchodníkovi stažení aktuálního kurzovního lístku DCC, pokud má tuto službu povolenou.

Data kurzovního lístku mohou mít různé formáty:
-	txt formát
-	xml formát 
#####           2.4.1.1.	Stažení aktuálního kurzovního lístku dcc/kurz
Metoda stáhne obchodníkovi aktuální kurzovní lístek z DCC enginu. Výstupní data kurzovního lístku budou konvertovány do formátu zadaného obchodníkem.
###     2.5. Pomocné funkce
####           2.5.1.	Kontrola dostupnosti echo
Metoda pouze ověří vzájemnou funkčnost a korektnost podpisů obou stran. V odpovědi vrací čas serveru internetového portálu.
## 3. Postup integrace, klíče a kde je vzít
pmAPI je přístupné z otevřeného internetu, proto je nutné data putující mezi systémy obchodníka a banky zabezpečit proti útokům. Komunikační kanál je zabezpečen protokolem SSL (HTTPS), pro ověření identity obchodníka jsou však navíc všechny požadavky odesílané na pmAPI podepsané jeho privátním klíčem a všechny odpovědi podepsané privátním klíčem banky.

pmAPI dostane k dispozici veřejnou část klíče, pomocí které může ověřit, že požadavek vygeneroval právě tento obchodník. Pro správnou funkčnost komunikace je tedy třeba vygenerovat tento pár privátní klíč + veřejný klíč. Privátní část předat do systému obchodníka (informační systém obchodníka komunikující na pmAPI) a veřejnou část předat na pmAPI, tedy bance. Tento proces je součástí integrace obchodníka.
###     3.1. První fáze
Integrace pmAPI začíná po sjednání poskytování těchto služeb v bance. Obchodník obdrží své MerchantID a uvede, ze které emailové adresy bude s bankou komunikovat.

**Postup integrace a spuštění ostrého provozu**

![Postup integrace a spuštění ostrého provozu](https://github.com/gbily/Test/blob/e8ab669caa1d226879c1924e5ec995ef9d9515a0/Postup%20integrace%20a%20spu%C5%A1t%C4%9Bn%C3%AD%20ostr%C3%A9ho%20provozu.png "Postup integrace a spuštění ostrého provozu")
###     3.2. Generování testovacích klíčů
Prostředky pro generování testovacích klíčů jsou dostupné na stránkách POSmerchant v integračním prostředí. Klíče se generují pomocí technologie JavaScript přímo na počítači obchodníka. Privátní část klíče se nepřenáší přes internet, banka k němu nemá přístup. Počítač by měl být bezpečný - nainstalována aktuální verze antivirového programu, dodržování pravidel bezpečného používání.

Generování a registrace testovacího klíče probíhá v POSMerchantu v integračním prostředí (https://iposman.iplatebnibrana.csob.cz/posmerchant) na záložce Obchodníci, po kliknutí na ikonu klíče. Tato funkce je dostupná až po sjednání a povolení funkce pmAPI.

![Generování klíčů](https://github.com/gbily/Test/blob/img/Generování%20klíčů.png "Generování klíčů")

Nabídne se možnost vygenerování testovacího klíče, případně vložení vlastního veřejného klíče.

![Generování klíčů obchodníka](https://github.com/gbily/Test/blob/img/Generov%C3%A1n%C3%AD%20kl%C3%AD%C4%8D%C5%AF%20obchodn%C3%ADka.png "Generování klíčů obchodníka")

Při zvolení možnosti vložit vlastní klíč, vloží obchodník veřejnou část svého RSA klíče (public key).

Při zvolení možnosti vygenerování testovacího klíče proběhne generování pomocí technologie JavaScript přímo na počítači obchodníka. Soukromý klíč neopustí počítač.

![Vygenerované klíče](https://github.com/gbily/Test/blob/img/Vygenerovan%C3%A9%20kl%C3%AD%C4%8De.png "Vygenerované klíče")

Veřejný klíč (public key) slouží k ověření podpisu zprávy od obchodníka na straně pmAPI. Tento klíč je vhodné stáhnout a zazálohovat pro případ jeho obnovy/opětovného zavedení na POSMerchant při problémech. Je nutné jej odeslat na server pomocí tlačítka POTVRDIT KLÍČE. Obchodník s tímto klíčem nepracuje. 

Soukromý klíč (private key) slouží k podpisu zprávy zaslané obchodníkem na pmAPI. Tento klíč je nutné stáhnout, bezpečně zazálohovat a předat do systému komunikujícího na pmAPI.

Po POTVRZENÍ KLÍČE je na registrovanou emailovou adresu distribuován aktivační kód pro potvrzení a aktivaci klíče.

![Aktivace obchodníka](https://github.com/gbily/Test/blob/img/Aktivace%20obchodn%C3%ADka.png "Aktivace obchodníka")

![Stáhnutí veřejného klíče](https://github.com/gbily/Test/blob/img/St%C3%A1hnut%C3%AD%20ve%C5%99ejn%C3%A9ho%20kl%C3%AD%C4%8De.png "Stáhnutí veřejného klíče")
###     3.3. Integrace
V tomto okamžiku může obchodník spustit integraci svého řešení (informačního systému) s pmAPI. Privátní klíč obchodníka společně s veřejným klíčem banky předá vývoji, který může vyvíjet a testovat proti veřejnému testovacímu prostředí pmAPI. 
####           3.3.1.	Integrační prostředí pmAPI
Pro integraci a otestování napojení obchodníkova informačního systému na pmAPI je obchodníkovi k dispozici veřejné integrační prostředí pmAPI.

V tomto prostředí je možné ověřit správnou implementaci všech podporovaných funkcí, podpisů požadavků a parsování odpovědí. Integrační prostředí nepracuje s reálnými transakcemi nebo výpisy, poskytuje pouze automaticky generované odpovědi, závislé na přijatém požadavku. Nicméně samotná funkcionalita je identická s produkčním prostředím.
###     3.4. Schválení
Jakmile je obchodník spokojen s výsledky integračních testů napojení svého informačního systému na testovací prostředí pmAPI, informuje banku o dokončení integračních testů a vůle zahájit přechod do ostrého provozu. Banka ověří výsledky testů a udělí obchodníkovi souhlas s přechodem do produkčního prostředí.

Ověření splnění všech integračních testů je dostupné v POSMerchantu v integračním prostředí (https://iposman.iplatebnibrana.csob.cz/posmerchant) na záložce Obchodníci, po kliknutí na ikonu fajfky. Tato funkce je dostupná až po sjednání a povolení funkce pmAPI.

![Schválení](https://github.com/gbily/Test/blob/img/Schv%C3%A1len%C3%AD.png "Schválení")

Zde je zobrazen seznam požadovaných volání pmAPI, které je doporučeno provést pro splnění integračních testů.

![Volání pmAPI](https://github.com/gbily/Test/blob/img/Vol%C3%A1n%C3%AD%20pmAPI.png "Volání pmAPI")
###     3.5. Generování ostrých klíčů
Po schválení přechodu do ostrého provozu je obchodníkovi umožněno vygenerovat nové, ostré klíče, které budou použity pro provoz v produkčním prostředí. Generátor ostrých klíčů je dostupný na stránkách POSmerchant. Generátor ostrých klíčů pracuje shodným způsobem, jako generátor testovacích klíčů, tedy lokálně na počítači obchodníka. 

Generování a registrace ostrého klíče probíhá v POSMerchantu (https://posman.csob.cz/posmerchant) na záložce Obchodníci, po kliknutí na ikonu klíče. Tato funkce je dostupná až po sjednání a povolení funkce pmAPI.

![Vygenerované ostré klíče](https://github.com/gbily/Test/blob/img/Vygenerovan%C3%A9%20ostr%C3%A9%20kl%C3%AD%C4%8De.png "Vygenerované ostré klíče")

Nabídne se možnost vygenerování klíče, případně vložení vlastního veřejného klíče.

![Generování klíčů ostrých klíčů obchodníka](https://github.com/gbily/Test/blob/img/Generov%C3%A1n%C3%AD%20kl%C3%AD%C4%8D%C5%AF%20ostr%C3%BDch%20kl%C3%AD%C4%8D%C5%AF%20obchodn%C3%ADka.png "Generování klíčů ostrých klíčů obchodníka")

Při zvolení možnosti vložit vlastní klíč, vloží obchodník veřejnou část svého RSA klíče (public key).

Při zvolení možnosti vygenerování klíče proběhne generování pomocí technologie JavaScript přímo na počítači obchodníka. Soukromý klíč neopustí počítač.

![Vygenerované ostré klíče](https://github.com/gbily/Test/blob/img/Vygenerovan%C3%A9%20ostr%C3%A9%20kl%C3%AD%C4%8De.png "Vygenerované ostré klíče")

Veřejný klíč (public key) slouží k ověření podpisu zprávy od obchodníka na strane pmAPI. Tento klíč je vhodné stáhnout a zazálohovat pro případ jeho obnovy/opětovného zavedení na POSMerchant při problémech. Je nutné jej odeslat na server pomocí tlačítka POTVRDIT KLÍČE. Obchodník s tímto klíčem nepracuje.

Soukromý klíč (private key) slouží k podpisu zprávy zaslané obchodníkem na pmAPI. Tento klíč je nutné stáhnout, bezpečně zazálohovat a předat do systému komunikujícího na pmAPI.

Po POTVRZENÍ KLÍČE je na registrovanou emailovou adresu distribuován aktivační kód pro potvrzení a aktivaci klíče.
###     3.6. Potvrzení ostrého klíče obchodníkem
Pro vyšší bezpečnost, ale také pro případ, kdy za běhu systému potřebuje obchodník svůj klíč vyměnit, je zde oproti testovacímu prostředí jeden krok navíc.

Obchodník má přístup do systému POSMerchant ČSOB, kde se jeho nově vygenerovaný klíč zobrazí v částí Platební terminály. Zde jej potvrdí a zaktivuje. K této operaci je třeba jednorázový aktivační kód, který obchodník obdržel na email při jeho generování, a který byl společně s veřejnou částí klíče odeslán na pmAPI. Okamžikem této aktivace se klíč aktivuje a pmAPI jej ihned začne používat. Tímto krokem je jednak dvojitě zabezpečeno předání klíče a současně obchodník sám určuje okamžik jeho zavedení.

![Aktivace obchodníka PP](https://github.com/gbily/Test/blob/img/Aktivace%20obchodn%C3%ADka%20PP.png "Aktivace obchodníka PP")

![Stáhnutí veřejného klíče PP](https://github.com/gbily/Test/blob/img/St%C3%A1hnut%C3%AD%20ve%C5%99ejn%C3%A9ho%20kl%C3%AD%C4%8De%20PP.png "Stáhnutí veřejného klíče PP")
###     3.7. Ostrý provoz
V tuto chvíli je již možné využívat veškeré funkce pmAPI.
## 4. Rozhraní pmAPI
###     4.1. Volání rozhraní pmAPI
pmAPI vychází z principů REST API, je dostupné přes HTTPS protokol, data jsou posílaná v JSON formátu.

Jednotlivé operace jsou implementované pomocí následujících HTTP metod:

| Metoda |                             Popis                            |
|--------|--------------------------------------------------------------|
| Post   | Získání dat a změna dat.                                     |
| Get    | Získání dat (Přenášené parametry musí být "URL enkódovány"). |

Poznámka:

Vzhledem k počtu parametrů, množství jejich kombinací a různorodosti jejich obsahu, byla pro získání dat zvolena HTTP metoda POST s JSON daty, místo standardní metody GET s daty v url.

V odpovědích na volání operací pmAPI jsou použity následující HTTP status kódy:

| Hodnota | Význam              | Popis                                                           |
|---------|---------------------|-----------------------------------------------------------------|
| 200     | OK                  | Požadavek byl úspěšný. Standardní odpověď.                      |
| 400     | Bad request         | Požadavek nemůže být vyřízen. Chyba syntaxe zápisu nebo adresy. |
| 403     | Forbidden           | Přístup odepřen.                                                |
| 404     | Not Found           | Zdroj nebyl nalezen.                                            |
| 405     | Method Not Allowed  | Požadovaná metoda není podporována.                             |
| 503     | Service Unavailable | Služba je dočasné mimo provoz                                   |

Při zpracování požadavku jsou nejprve kontrolovány základní parametry a ověřen podpis požadavku. V případě chyby při této základní kontrole obsahuje odpověď z hlediska bezpečnosti pouze obecný HTTP status kód (např. 400 Bad Request nebo 403 Forbidden).
###     4.2. Podepisování
Veškeré požadavky zaslané na pmAPI musí být podepsány soukromým klíčem obchodníka a jsou při přijetí ověřeny veřejným klíčem obchodníka.

Veškeré odpovědi od pmAPI jsou podepsány soukromým klíčem banky a měly by být při přijetí obchodníkem ověřeny veřejným klíčem banky.
####           4.2.1.	Sestavení podpisu volání
Z dat odesílaných na server sestavíme RETEZEC_ZPRAVY tak, že jednotlivé datové položky seřadíme za sebe v pořadí, v jakém jsou ve specifikaci uvedeny. Pro oddělení jednotlivých položek se použije oddělovač „|“. Do výpočtu zahrneme všechny parametry odeslané v požadavku. Pokud tedy nebude některý nepovinný parametr použit, nebude ani ve výsledném řetězci.

Pokud je položkou zprávy vnořený datový objekt, prostupuje se položkami tohoto objektu. V případě seznamu se do výsledného řetězce vkládají položky ve stejném pořadí, v jakém jsou uvedeny ve zprávě.

Čísla jsou reprezentována v ASCII podobě, znaky jsou zapisované ve své binární reprezentaci (nejsou povoleny entity \uXXXX).
Pro samotný výpočet podpisu se pak použije privátní klíč obchodníka.

Pro samotný výpočet podpisu se pak použije privátní klíč obchodníka.

`Signature = BASE64_ENCODE(RSA_SIGN(RETEZEC_ZPRAVY))`

Příklad:
```
RETEZEC_ZPRAVY = "M1TEST0001|20160215|12050|87598547|F05334EE|1789600"
RSA_SIGNATURE = RSA_SIGN(RETEZEC_ZPRAVY)
base64-encoded-signature = BASE64_ENCODE(RSA_SIGNATURE)
Zpráva = {
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "Amount":12050,
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "Signature":"base64-encoded-signature"
}
```
V případě ověření podpisu na POSmerchant API u GET operací do podpisu vstupují hodnoty parametrů, které jsou „URL dekódovány“.

Pro výpočet podpisu (RSA_SIGN) je potřeba použít algoritmus založený na SHA-1. Například v Javě je potřeba použít při inicializaci třídy `java.security.Signature` algoritmus "SHA1withRSA", v PHP je potřeba použít "OPENSSL_ALGO_SHA1", což je defaultní algoritmus pro funkce `openssl_sign()` a `openssl_verify()`.

#####           4.2.1.1.	Příklad sestavení podpisu pro požadavek posílaný pomocí metody POST
V operaci pro zjištění stavu transakce payment/state podepisujeme parametry následujícím způsobem.

**Požadovaný výstup**
```
curl -v –X POST https://posman.csob.cz/api/pmapi/v1.0/payment/state \
-H "Content-Type:application/json" \
-d '{
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "Amount":12050,
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "Signature":"base64-encoded-signature"
}'
```
**Výpočet Signature**
```
RETEZEC_ZPRAVY = "M1TEST0001|20160215|12050|87598547|F05334EE|1789600"
Signature = BASE64_ENCODE(RSA_SIGN(RETEZEC_ZPRAVY))
```
#####           4.2.1.2.	Příklad sestavení podpisu pro požadavek posílaný pomocí metody GET
Přenášené parametry v URL musí být „URL enkódovány“.

V operaci pro stažení konkrétního výpisu report/get podepisujeme parametry následujícím způsobem.

**Požadovaný výstup**
```
curl -v -X GET https://posman.csob.cz/api/pmapi/v1.0/report/get/12547/458849/00001.zip/url-encoded-base64-encoded-signature
```
**Výpočet Signature**
```
RETEZEC_ZPRAVY = "12547|458849|00001.zip"
Signature = BASE64_ENCODE(RSA_SIGN(RETEZEC_ZPRAVY))
```
####           4.2.2.	Ověření podpisu odpovědi
Podobně jako u vytvoření podpisu požadavku se pro ověření podpisu odpovědi z jednotlivých položek odpovědi vytvoří RETEZEC_ZPRAVY a pro ověření podpisu se použije veřejný klíč mpAPI.
#####           4.2.2.1.	Příklad sestavení podpisu pro odpověď
V operaci pro stažení konkrétního výpisu report/get vypočítáme podpis následujícím způsobem.

**Odpověď**
```
{`
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "Amount":12050,
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "ResultCode":0,
  "ResultMessage":"OK",
  "DateTime":"20160215132045",
  "PayStatus":"authorized",
  "Signature":"base64-encoded-signature"
}
```
**Výpočet Signature**
```
RETEZEC_ZPRAVY = "M1TEST0001|20160215|12050|87598547|F05334EE|1789600|0|OK|20160215132045|authorized"
Signature = BASE64_ENCODE(RSA_SIGN(RETEZEC_ZPRAVY))
```
###     4.3. Podepisování
####           4.3.1.	Transakce
#####           4.3.1.1.	Zjištění stavu transakce payment/state
HTTP metoda: **POST**

Formát dat: **JSON**

**Požadavek**

Požadavky pro payment/state, payment/reverse a payment/process obsahují jednotnou sadu parametrů popisující transakci. U jednotlivých volání funkcí jsou již uvedeny pouze parametry rozšiřující tuto společnou definici.

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| PosId     | String | Identifikátor platebního terminálu. 10 znaků. |
| Date      | String | Datum vzniku transakce ve formátu YYYYMMDD.   |
| Amount    | Number | Částka transakce v setinách základní měny.    |
| VarSymbol | String | Variabilní symbol.                            |
| AuthCode  | String | Autorizační kód.                              |
| SeqId     | String | Sekvenční id transakce.                       |
| Signature | String | Podpis požadavku, kódováno v BASE64.          |

**Struktura dat podpisu požadavku:**

`PosId|Date|Amount|VarSymbol|AuthCode|SeqId`

Příklad volání:
```
curl -v –X POST https://posman.csob.cz/api/pmapi/v1.0/payment/state \
-H "Content-Type:application/json" \
-d '{
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "Amount":12050,
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "Signature":"base64-encoded-signature"
}
```
**Návratové hodnoty**

Návratové hodnoty pro payment/state, payment/reverse a payment/process obsahují jednotnou sadu parametrů, popisující aktuální transakci, její stav a výsledek operace. U jednotlivých volání funkcí jsou již uvedeny pouze příklady návratových hodnot s odkazem na tuto společnou definici.

| Položka       | Typ    | Popis                                                   |
|---------------|--------|---------------------------------------------------------|
| PosId         | String | Identifikátor platebního terminálu. 10 znaků.           |
| Date          | String | Datum vzniku transakce ve formátu YYYYMMDD.             |
| Amount        | Number | Částka transakce v setinách základní měny.              |
| Currency      | String | Měna.                                                   |
| VarSymbol     | String | Variabilní symbol.                                      |
| AuthCode      | String | Autorizační kód.                                        |
| SeqId         | String | Sekvenční id transakce.                                 |
| ResultCode    | Number | Výsledek operace, viz výčet.                            |
| ResultMessage | String | Textový popis výsledku operace.                         |
| DateTime      | String | Datum a čas vzniku transakce ve formátu YYYYMMDDHHMMSS. |
| PayStatus     | String | Stav transakce.                                         |
| Signature     | String | Podpis požadavku, kódováno v BASE64.                    |

Návratové kódy:

| ResultCode | ResultMessage            | Popis                              |
|------------|--------------------------|------------------------------------|
| 0          | OK                       | Operace proběhla korektně.         |
| 100        | Missing parameter {name} | Chybějící povinný parametr.        |
| 110        | Invalid parameter {name} | Chybný formát parametru.           |
| 120        | Payment not found        | Platba nenalezena.                 |
| 130        | Payment not unique       | Transakce není jednoznačně určena. |
| 140        | Operation not allowed    | Nepovolená operace.                |

**Struktura dat podpisu odpovědi:**

PosId|Date|Amount|VarSymbol|AuthCode|SeqId|ResultCode|ResultMessage|DateTime|PayStatus

**Příklad návratových hodnot pro úspěšně zpracovaný požadavek:**
```
{
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "Amount":12050,
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "ResultCode":0,
  "ResultMessage":"OK",
  "DateTime":"20160215132045",
  "PayStatus":"authorized",
  "Signature":"base64-encoded-signature"
}
```
**Příklad návratových hodnot pro neúspěšně zpracovaný požadavek:**
```
{
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "ResultCode":100,
  "ResultMessage":"Missing parameter 'Amount'",
  "Signature":"base64-encoded-signature"
}
```
#####           4.3.1.2.	Reverzování transakce payment/reverse
HTTP metoda: **POST**

Formát dat: **JSON**
 
**Požadavek**

Základní parametry jsou shodné s definicí popsanou u operace payment/state.

| Položka         | Typ    | Popis                                                   |
|-----------------|--------|---------------------------------------------------------|
| ProcessedAmount | Number | Částka transakce v setinách základní měny.              |

**Struktura dat podpisu volání:**

Struktura je shodná jako u operace payment/state doplněna o hodnotu |ProcessedAmount.

**Příklad volání:**

```
curl -v –X POST https://posman.csob.cz/api/pmapi/v1.0/payment/reverse \
-H "Content-Type:application/json" \
-d '{
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "Amount":12050,
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "ProcessedAmount":"2000",
  "Signature":"base64-encoded-signature"
}
```
**Návratové hodnoty**

Návratové hodnoty jsou shodné s definicí popsanou u operace payment/state.

Struktura dat pro podpis odpovědi je shodná jako u operace payment/state.

**Příklad návratových hodnot pro úspěšně zpracovaný požadavek:**
```
{
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "Amount":12050,
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "ResultCode":0,
  "ResultMessage":"OK",
  "DateTime":"20160215132045",
  "PayStatus":"authorized",
  "Signature":"base64-encoded-signature"
}
```
**Příklad návratových hodnot pro neúspěšně zpracovaný požadavek:**
```
{
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "ResultCode":100,
  "ResultMessage":"Missing parameter 'Amount'",
  "Signature":"base64-encoded-signature"
}
```
#####           4.3.1.3.	Znovuproplacení transakce payment/process
HTTP metoda: **POST**
Formát dat: **JSON**

**Požadavek**

Základní parametry jsou shodné s definicí popsanou u operace payment/state.

| Položka         | Typ    | Popis                                                   |
|-----------------|--------|---------------------------------------------------------|
| ProcessedAmount | Number | Částka transakce v setinách základní měny.              |

Podpis volání je shodný s operací payment/reverse.

**Příklad volání:**
```
curl -v –X POST https://posman.csob.cz/api/pmapi/v1.0/payment/reverse \
-H "Content-Type:application/json" \
-d '{
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "Amount":12050,
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "ProcessedAmount":"2000",
  "Signature":"base64-encoded-signature"
}'
```
**Návratové hodnoty**

Návratové hodnoty jsou shodné s definicí popsanou u operace payment/state.

Podpis odpovědi je shodný s operací payment/reverse.

**Příklad návratových hodnot pro úspěšně zpracovaný požadavek:**
```
{
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "Amount":12050,
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "ResultCode":0,
  "ResultMessage":"OK",
  "DateTime":"20160215132045",
  "PayStatus":"authorized",
  "Signature":"base64-encoded-signature"
}
```
**Příklad návratových hodnot pro neúspěšně zpracovaný požadavek:**
```
{
  "PosId":"M1TEST0001",
  "Date":"20160215",
  "VarSymbol":"87598547",
  "AuthCode":"F05334EE",
  "SeqId":"1789600",
  "ResultCode":100,
  "ResultMessage":"Missing parameter 'Amount'",
  "Signature":"base64-encoded-signature"
}
```
####           4.3.2.	Výpisy
Pro přístup ke konkrétnímu výpisu je nutné zadat jeho unikátní ReportId. Tento identifikátor je uveden v POSMerchantu (https://posman.csob.cz/posmerchant), na záložce Obchodníci – Výpisy.

![Výpisy](https://github.com/gbily/Test/blob/img/V%C3%BDpisy.png "Výpisy")

![Detail výpisů](https://github.com/gbily/Test/blob/img/Detail%20v%C3%BDpis%C5%AF.png "Detail výpisů")
#####           4.3.2.1.	Seznam vygenerovaných výpisů report/list
HTTP metoda: **POST**

Formát dat: **JSON**

**Požadavek**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| MerchantId     | String | Identifikátor obchodníka. |
| ReportId      | String | Identifikátor výpisu.   |
| Date    | String | Datum ve formátu YYYYMMDD.    |
| Signature | String | Podpis požadavku, kódováno v BASE64.         |

**Struktura dat podpisu volání:**

MerchantId|ReportId|Date

**Příklad volání:**
```
curl -v –X POST https://posman.csob.cz/api/pmapi/v1.0/report/list \
-H "Content-Type:application/json" \
-d '{
  "MerchantId":"12547",
  "ReportId":"458849",
  "Date":"20160215",
  "Signature":"base64-encoded-signature"
}'
```
**Návratové hodnoty**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| MerchantId     | String | Identifikátor obchodníka. |
| ReportId      | String | Identifikátor výpisu.   |
| Date    | String | Datum ve formátu YYYYMMDD.    |
| ResultCode    | Number | Výsledek operace, viz. výčet.    |
| ResultMessage    | String | Textový popis výsledku operace.    |
| Files    | Object | Seznam názvů souborů výpisů.    |
| Signature | String | Podpis požadavku, kódováno v BASE64.         |

Report:

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| Name     | String | Název výpisu. |

Návratové kódy:

| ResultCode | ResultMessage            | Popis                              |
|------------|--------------------------|------------------------------------|
| 0          | OK                       | Operace proběhla korektně.         |
| 100        | Missing parameter {name} | Chybějící povinný parametr.        |
| 110        | Invalid parameter {name} | Chybný formát parametru.           |
| 120        | Merchant not found        | Obchodník nenalezen.                |
| 130        | Report not found      | Výpis nenalezen. |

**Struktura dat pro podpis odpovědi:**

MerchantId|ReportId|Date|ResultCode|ResultMessage|[Files]*

Files:

Name|Dir|Extension|Length|LastModified

**Příklad návratových hodnot pro úspěšně zpracovaný požadavek:**
```
{
  "MerchantId":"12547",
  "ReportId":"458849",
  "Date":"20160215",
  "ResultCode":0,
  "ResultMessage":"OK",
  "Files":[
    {
      "name": "00001.zip",
    },
    {
      "name": "00002.zip",
    },
    {
      "name": "00003.zip",
    }
  ],
  "Signature":"base64-encoded-signature"
}
```
**Příklad návratových hodnot pro neúspěšně zpracovaný požadavek:**
```
{
  "MerchantId":"12547",
  "Date":"20160215",
  "ResultCode":100,
  "ResultMessage":"Missing parameter 'ReportId'",
  "Signature":"base64-encoded-signature"
}
```
#####           4.3.2.2.	Stažení konkrétního výpisu report/get
HTTP metoda: **GET**
Formát dat: **URL**
 
**Požadavek**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| MerchantId     | String | Identifikátor obchodníka. |
| ReportId      | String | Identifikátor výpisu.   |
| Dir    | String | Název složky.    |
| File    | String | Název souboru výpisu.    |
| Signature | String | Podpis požadavku, kódováno v BASE64.         |

**Struktura dat podpisu volání:**

MerchantId|ReportId|Dir|File

**URL**
```
https://posman.csob.cz/api/pmapi/v1.0/report/get/{MerchantId}/{ReportId}/{Dir}/{File}/{Signature}
```
**Příklad volání:**
```
curl -v -X GET https://posman.csob.cz/api/pmapi/v1.0/report/get/12547/458849/0001/00001.zip/url-encoded-base64-encoded-signature
```
**Návratové hodnoty**
Standardní HTTP status kódy.

Soubor výpisu.
####           4.3.3.	Account Transport
#####           4.3.3.1.	Seznam tapů transport/tap/list
HTTP metoda: **POST**

Formát dat: **JSON**

**Požadavek**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| MerchantId     | String | Identifikátor obchodníka. |
| DateFrom      | String | Datum od ve formátu YYYYMMDD.   |
| DateTo    | String | Datum do ve formátu YYYYMMDD.    |
| Vs    | String | Variabilní symbol.    |
| Token | String | Token.        |
| Status | Number | Status tapu.      |
| ResponseCode | Number | Návratový kód terminálu.     |
| TerminalId | String | Identifikátor terminálu.        |
| Signature | String | Podpis požadavku, kódováno v BASE64.       |

**Struktura dat podpisu volání:**

MerchantId|DateFrom|DateTo

*Příklad volání:*
```
curl -v –X POST https://posman.csob.cz/api/pmapi/v1.0/transport/tap/list \
-H "Content-Type:application/json" \
-d '{
  "MerchantId":"12547",
  "DateFrom":"20160205",
  "DateTo":"20160215",
  "Vs":"12547",
  "Token":"ABCDE12345",
  "Status":"1",
  "ResponseCode":"2",
  "TerminalId":"12345", 
  "Signature":"base64-encoded-signature"
}'
```
**Návratové hodnoty**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| MerchantId     | String | Identifikátor obchodníka. |
| DateFrom      | String | Datum od ve formátu YYYYMMDD.   |
| DateTo    | String | Datum do ve formátu YYYYMMDD.    |
| Vs    | String | Variabilní symbol.    |
| Token | String | Token.        |
| Status | Number | Status.      |
| ResponseCode | Number | Návratový kód terminálu.     |
| TerminalId | String | Identifikátor terminálu.        |
| ResultCode | Number | Výsledek operace. Viz výčet       |
| ResultMessage | String | Textový popis výsledku operace.       |
| TapList | Object | Seznam nalezených tapů.       |
| Signature | String | Podpis požadavku, kódováno v BASE64.       |

**TapList**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| TapDttm     | String | Datum a čas tapu ve formátu DD.MM.YYYY HH:MM:SS |
| TerminalId      | String | Identifikátor terminálu.   |
| TerminalStatus    | Integer | Status odbavení na terminálu.    |
| TerminalResponse    | Integer | Návratový kód, který vydal terminál při odbavení karty.   |
| Token | String | Karetní token.       |
| MaskedPan | String | Maskované číslo karty.     |
| Vs | String | Variabilní symbol tapu z terminálu.     |
| ValidatorId | String | Identifikátor validátoru.       |
| TxCounter | Integer | Pořadové číslo transakce z validátoru.       |
| Amount | Long | Částka jízdenky.    |
| Vat | Integer | Daň.      |
| LineId | String | Číslo linky.     |
| VehicleId | VehicleId | Číslo spoje.   |
| EntryStopTariffNumber | String | Tarifní (pořadové) číslo výchozí zastávky.      |
| EntryStopNumber | String | Evidenční číslo zastávky.    |
| EntryStopZoneNumber | String | Číslo zóny výchozí zastávky.      |
| ExitStopZoneNumber | String | Číslo zóny cílové zastávky.    |

**Návratové kódy:**

| ResultCode | ResultMessage            | Popis                              |
|------------|--------------------------|------------------------------------|
| 0          | OK                       | Operace proběhla korektně.         |
| 100        | Missing parameter {name} | Chybějící povinný parametr.        |
| 110        | Invalid parameter {name} | Chybný formát parametru.           |
| 160        | Merchant not found        | Obchodník nenalezen.                |
| 180        | Date out of range      | Datum poslední kontroly přesáhl 1 měsíc od aktuálního data. |
| 190        | Taps not found      | Tapy nenalezeny. |

**Struktura dat pro podpis odpovědi:**

MerchantId|DateFrom|DateTo|ResultCode|ResultMessage|[TapList]*

TapList:

TerminalId|TerminalStatus|TerminalResponse|StoplistVersion|Token|MaskedPan|Vs|TapDttm|ValidatorId|TxCounter|Amount|Vat|LineId|VehicleId|EntryStopTariffNumber|EntryStopNumber|EntryStopZoneNumber|ExitStopZoneNumber

**Příklad návratových hodnot pro úspěšně zpracovaný požadavek:**
```
{
  "MerchantId":"12547",
  "DateFrom":"20160205",
  "DateTo":"20160215",
  "ResultCode":0,
  "ResultMessage":"OK",
  "TapList":[
    {
       "TapDttm"	  : "20.07.2016 21:24:35",
"TerminalId" 	  : "145456",
"TerminalStatus": 1,
"TerminalResponse": 1,
"StopListVersion": "124",
"Token"	  : "4546456",
"Vs" 		  : "54878124",
"ValidatorId"	  : "874",
"TxCounter"	  : 2,
"Amount"	  : 10000,
"Vat"	 	  : 20,
"LineId" 	  : "132",
"VehicleId"	  : "324",
"EntryStopTariffNumber": "3",
"EntryStopNumber": "5",
"EntryStopZoneNumber": "2",
"ExitStopZoneNumber": "4"
    },
     {
       "TapDttm"	  : "12.08.2016 13:52:31",
"TerminalId" 	  : "145456",
"TerminalStatus": 2,
"TerminalResponse": 1,
"StopListVersion": "125",
"Token"	  : "4546478",
"Vs" 		  : "54878125",
"ValidatorId"	  : "874",
"TxCounter"	  : 3,
"Amount"	  : 20000,
"Vat"	 	  : 20,
"LineId" 	  : "132",
"VehicleId"	  : "324",
"EntryStopTariffNumber": "7",
"EntryStopNumber": "10",
"EntryStopZoneNumber": "1",
"ExitStopZoneNumber": "2"
    }
  ],
  "Signature":"base64-encoded-signature"
}
```
**Příklad návratových hodnot pro neúspěšně zpracovaný požadavek:**
```
{
  "DateFrom":"20160205",
  "DateTo":"20160215",
  "ResultCode":100,
  "ResultMessage":"Missing parameter MerchantId",
  "Signature":"base64-encoded-signature"
}
```
#####           4.3.3.2.	Seznam clearingových transakcí transport/clear/list
HTTP metoda: **POST**

Formát dat: **JSON**

**Požadavek**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| MerchantId     | String | Identifikátor obchodníka. |
| DateFrom      | String | Datum od ve formátu YYYYMMDD.   |
| DateTo    | String | Datum do ve formátu YYYYMMDD.    |
| Vs    | String | Variabilní symbol.    |
| Token | String | Token.        |
| Amount | Number | Zastropovaná částka.      |
| Signature | String | Podpis požadavku, kódováno v BASE64.       |

**Struktura dat podpisu volání:**

MerchantId|DateFrom|DateTo

*Příklad volání:*
```
curl -v –X POST https://posman.csob.cz/api/pmapi/v1.0/transport/clear/list \
-H "Content-Type:application/json" \
-d '{
  "MerchantId":"12547",
  "DateFrom":"20160205",
  "DateTo":"20160215",
  "Vs":"10500",
  "Token":"ABCDE12345",
  "Amount":"10200",
  "Signature":"base64-encoded-signature"
}'
```
**Návratové hodnoty**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| MerchantId     | String | Identifikátor obchodníka. |
| DateFrom      | String | Datum od ve formátu YYYYMMDD.   |
| DateTo    | String | Datum do ve formátu YYYYMMDD.    |
| Vs    | String | Variabilní symbol.    |
| Token | String | Token.        |
| Status | Number | Status.      |
| ResultCode | Number | Výsledek operace. Viz výčet       |
| ResultMessage | String | Textový popis výsledku operace.       |
| ClearList | Object | Seznam nalezených clearingových transakcí.       |
| Signature | String | Podpis požadavku, kódováno v BASE64.       |

**ClearList**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| Token | String | Karetní token.       |
| MaskedPan | String | Maskované číslo karty.     |
| AmountBase | Long | Částka základní (součet všech tapů)     |
| AmountFinal | Long | Částka zastropovaná (snížená částka na denní jízdné).      |
| FirstTapDttm | String | Datum prvního tapu ve formátu DD.MM.YYYY HH:MM:SS.      |
| LastTapDttm | String | Datum zúčtování ve formátu DD.MM.YYYY HH:MM:SS.   |
| PayDttm | String | Číslo zóny výchozí zastávky.      |
| Vs | String | Variabilní symbol.   |

**Návratové kódy:**

| ResultCode | ResultMessage            | Popis                              |
|------------|--------------------------|------------------------------------|
| 0          | OK                       | Operace proběhla korektně.         |
| 100        | Missing parameter {name} | Chybějící povinný parametr.        |
| 110        | Invalid parameter {name} | Chybný formát parametru.           |
| 160        | Merchant not found        | Obchodník nenalezen.                |
| 180        | Date out of range      | Datum poslední kontroly přesáhl 1 měsíc od aktuálního data. |
| 200        | Clears not found      | Clearingové transakce nenalezeny nenalezeny. |

**Struktura dat podpisu odpovědi:**

MerchantId|DateFrom|DateTo|ResultCode|ResultMessage|[ClearList]*

ClearList:

Token|MaskedPan|AmountBase|AmountFinal|FirstTapDttm|LastTapDttm|PayDttm|Vs

**Příklad návratových hodnot pro úspěšně zpracovaný požadavek:**
```
{
  "MerchantId":"12547",
  "DateFrom":"20160205",
  "DateTo":"20160215",
  "ResultCode":0,
  "ResultMessage":"OK",
  "ClearList":[
    {
"Token" 	: "145456",
"MaskedPan"	: "145446****4545",
"AmountBase" : 10000,
"AmountFinal": 8000,
"FirstTapDttm": "08.01.2016 12:51:28",
"LastTapDttm" : "10.01.2016 02:15:35",
"PayDttm"	 : "10.04.2016 02:25:35",
"Vs" 		  : "54878124"
    },
     {
"Token" 	: "145456",
"MaskedPan"	: "921646****4975",
"AmountBase" : 20000,
"AmountFinal": 16000,
"FirstTapDttm": "20.07.2017 14:24:45",
"LastTapDttm" : "20.07.2017 21:01:35",
"PayDttm"	 : "20.07.2017 22:15:35",
"Vs" 		  : "42378124"
    }
  ],
  "Signature":"base64-encoded-signature"
}
```
**Příklad návratových hodnot pro neúspěšně zpracovaný požadavek:**
```
{
  "DateFrom":"20160205",
  "DateTo":"20160215",
  "ResultCode":100,
  "ResultMessage":"Missing parameter MerchantId",
  "Signature":"base64-encoded-signature"
}
```
####           4.3.4.	DCC
#####           4.3.4.1.	Aktuální kurzovní lístek dcc/kurz
HTTP metoda: **POST**

Formát dat: **JSON**
 
**Požadavek**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| MerchantId     | String | Identifikátor obchodníka. |
| OutputFormat      | String | Formát výstupních dat kurzovního lístku.  |
| Signature | String | Podpis požadavku, kódováno v BASE64.       |

**Struktura dat podpisu volání:**

MerchantId|OutputFormat

*Příklad volání:*
```
curl -v –X POST https://posman.csob.cz/api/pmapi/v1.0/dcc/kurz \
-H "Content-Type:application/json" \
-d '{
  "MerchantId":"12547",
  "OutputFormat":"XML",
  "Signature":"base64-encoded-signature"
}'
```
**Návratové hodnoty**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| MerchantId     | String | Identifikátor obchodníka. |
| OutputFormat      | String | Formát výstupních dat kurzovního lístku.  |
| Name     | String | Název výstupního souboru kurzovního lístku. |
| KurzListek     | String | Data kurzovního lístku ve výstupním formátu. |
| Signature | String | Podpis požadavku, kódováno v BASE64.       |

**Struktura dat podpisu odpovědi:**

MerchantId|OutputFormat|ResultCode|ResultMessage|Name|KurzListek

Popis dat kurzovního lístku, podle daných formátů:

Textový formát:

*textový kód vstupní měny;číselný kód vstupní měny;textový kód výstupní měny;číselný kód výstupní měny;částka za jednotku vstupní měny;datum platnosti od;datum a čas platnosti do*

XML formát:
```
<DCCKL>
<ITEM>
<CURRENCY_NUM_SRC> textový kód vstupní měny </CURRENCY_TXT_SRC> 
<CURRENCY_NUM_SRC> číselný kód vstupní měny </CURRENCY_NUM_SRC>
      <CURRENCY_TXT_DST> textový kód výstupní měny </CURRENCY_TXT_DST>
      <CURRENCY_NUM_DST> číselný kód výstupní měny </CURRENCY_NUM_DST>
      <AMOUNT> částka za jednotku vstupní měny </AMOUNT>
      <VALID_FROM> datum platnosti od </VALID_FROM>
      <VALID_TO> datum a čas platnosti do *</VALID_TO>
</ITEM>
… 
…
</DCCKL>
```
* Datum a čas platnosti do, je 24 hodin po čase žádosti o kurzovní lístek.

**Návratové kódy:**

| ResultCode | ResultMessage            | Popis                              |
|------------|--------------------------|------------------------------------|
| 0          | OK                       | Operace proběhla korektně.         |
| 100        | Missing parameter {name} | Chybějící povinný parametr.        |
| 110        | Invalid parameter {name} | Chybný formát parametru.           |
| 160        | Merchant not found        | Obchodník nenalezen.                |
| 180        | Date out of range      | Datum poslední kontroly přesáhl 1 měsíc od aktuálního data. |
| 210        | DCC kurz not found      | Kurzovní lístek nebyl nalezen. |

**Příklad návratových hodnot pro úspěšně zpracovaný požadavek:**
```
{
  "MerchantId":"12547",
  "OutputFormat":"XML",
  "ResultCode":0,
  "ResultMessage":"OK",
  "Name":"DCC_06092018_f2bc5.xml",
  "KurzListek":
"<DCCKL>
  <ITEM>
      <CURRENCY_TXT_SRC>DKK</CURRENCY_TXT_SRC>
      <CURRENCY_NUM_SRC>208</CURRENCY_NUM_SRC>
      <CURRENCY_TXT_DST>CZK</CURRENCY_TXT_DST>
      <CURRENCY_NUM_DST>203</CURRENCY_NUM_DST>
      <AMOUNT>0.2928871</AMOUNT>
      <VALID_FROM>2018-09-05</VALID_FROM>
      <VALID_TO>2018-09-06 06:35:04</VALID_TO>
  </ITEM>
  <ITEM>
      <CURRENCY_TXT_SRC>EUR</CURRENCY_TXT_SRC>
      <CURRENCY_NUM_SRC>978</CURRENCY_NUM_SRC>
      <CURRENCY_TXT_DST>CZK</CURRENCY_TXT_DST>
      <CURRENCY_NUM_DST>203</CURRENCY_NUM_DST>
      <AMOUNT>27.07109</AMOUNT>
      <VALID_FROM>2018-09-05</VALID_FROM>
      <VALID_TO>2018-09-06 06:35:04</VALID_TO>
  </ITEM>
</DCCKL>",
  "Signature":"base64-encoded-signature"
}
```
**Příklad návratových hodnot pro neúspěšně zpracovaný požadavek:**
```
{
  "ResultCode":100,
  "ResultMessage":"Missing parameter MerchantId",
  "Signature":"base64-encoded-signature"
}
```
####           4.3.5.	Pomocné funkce
#####           4.3.5.1.	Kontrola dostupnosti echo
HTTP metoda: **GET**

**Požadavek**

Požadavek nemá žádné parametry.

**URL**

`https://posman.csob.cz/api/pmapi/v1.0/echo`

**Příklad volání:**

`curl -v -X GET https://posman.csob.cz/api/pmapi/v1.0/echo`

**Návratové hodnoty**

| Položka   | Typ    | Popis                                         |
|-----------|--------|-----------------------------------------------|
| Date     | String | Datum ve formátu YYYYMMDDHHMMSS. |

**Příklad návratových hodnot:**

`20160217154501`
