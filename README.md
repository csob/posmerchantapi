# POSMerchant API
Je **rozhraní mezi Internetovým portálem ČSOB POSMerchant a informačním systémem klienta ČSOB** (dále jen pmAPI). Internetový portál ČSOB POSMerchant je určen pro klienty ČSOB, kteří využívají její služby zejména v oblasti platebních terminálů a platební brány. Portál má dvě základní funkce - poskytnout klientovi **náhled na transakce** a umožnit **změnu jejich stavu**.

**pmAPI** zpřístupňuje tyto klíčové funkce klientům prostřednictvím **rest služeb**. Komunikace mezi informačním systémem klienta a portálem ČSOB probíhá přes šifrovaný protokol HTTPS a dále je zabezpečena pomocí elektronických podpisů všech požadavků a odpovědí.
## 1. Slovníček nejdůležitějších pojmů
V kapitole jsou vysvětleny nejčastěji používáné zkratky a pojmy dále v dokumentaci.
## 2. Funkce pmAPI
Zde jsou popsány jednotlivé funkce pmAPI, které se týkají **transakcí** (např. změna jejich stavu), **výpisů** (stažení, seznam), **zúčtovacího modulu** (Account Transport - pro odbavení bank. karet ve veřejné dopravě), **DCC** (kurzovní lístek) a další **pomocné funkce** (echo).
## 3. Postup integrace, klíče a kde je vzít
Komunikační kanál je zabezpečen protokolem SSL (HTTPS), pro ověření identity obchodníka jsou však navíc všechny požadavky odesílané na pmAPI podepsané jeho privátním klíčem a všechny odpovědi podepsané privátním klíčem banky.

Jednotlivé fáze a postup je dále rozepsán v této kapitole.
## 4. Rozhraní pmAPI
Popisuje **volání rozhraní pmAPI**, které vychází z principů REST API, je dostupné přes HTTPS protokol a data jsou posílaná v JSON formátu. Dále **podepisování**, tj., že každý požadavek i odpověď musí být podepsány soukromým klíčem obchodníka/banky a měly by být při přijetí obchodníkem ověřeny veřejným klíčem obchodníka/banky. Jsou zde také uvedeny příklady volání jednotlivých metod a jejich návratové hodnoty.
