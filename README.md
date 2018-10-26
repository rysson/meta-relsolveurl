# meta-relsolveurl
Kodi addon. Meta url resolver

## Szkic


#### Cel

Najpierw wyjaśnię

> można zostawić w tranzytowym dodatku API jak jest teraz,

Chodzi o to, żeby meta-resolver (a nie tranzytowy, przepraszam) był widziany nawet przez obecne wtyczki tak samo jak obecny resolveurl (czyli punkt wejścia lib.resolveurl). Dzięki temu wtyczki, których nie mamy w naszych łapach też będą działać.


#### Meta-resolver

Teraz dalej, zrodzony wczoraj pomysł meta resolvera to tylko pośrednik do właściwych resolverów. Z zewnętrznymi regułami priorytetów (to pomysł z dzisiaj). 

Oryginalny resolveurl od jsergio paczkujemy jako script.module.resolveurl.jsergio (5.0.17).
Dodatkowo zmieniamy punkt wejścia (nazwę lib.resolveurl na lib.resolveurl-jsergio albo coś podobnego).

resolveurl-6.4-jsergio5.0.17  (*to jest właściwy pakiet tranzytowy*) \
↳meta-resolveurl-0.1.0        (*wymaga*¹) \
↳resolveurl.jsergio-5.0.17    (*wymaga*¹) \
↳(*wymaga¹ jakiegoś naszego pakietu z preferencjami, patrz dalej*)

meta-resolveurl-0.1.0           (*to on ma główny punkt wejścia: `lib.resolverurl`) \
↳ resolveurl.jsergio-5.0.17     (*łączy się*²) \
↳ resolveurl.pl 0.1.0           (*łączy się*²) \
↳ resolveurl.youtubedl-0.1.0    (*łączy się*²) \
↳ ...

resolveurl.youtubedl-0.1.0 \
↳ youtubedl-2018.10.05.0  (*wymaga*¹, *to tylko biblioteka*)

... (*itd.*)

Tak jak wspomniał notoco, wszyscy powinnismy przejść (w zależnościach) na meta-resolver.
Z tym, że jak ktoś się zagapi to też będzie działać (zainstaluje się meta-resolver + resolveurl.jsergio).

Podział na repozytoria popieram. Choć z powyższym podziałem na pakiety powinno się to dać upchnąć nawet w pojedynczym binarnym repo.

¹) Wymaga, czyli jest w zależnościach (`requires/import`) \
²) Łączy się, czyli nie ma w zależnościach ale jest zainstalowany to używa. Innymi słowy przeszukuje zainstalowane dodatki pod kątem sub-resolv


#### Paczki preferencji

Do tych wszystkich naszych resolverów (pl, youdubedl itd.) można dodać jakąś klasę bazową (rodzaj wzorca), dzięki czemu pisanie ich będzie banalne. To już szczegół techniczny, choć może się wtedy pojawić zależność od powiedzmy script.module.resolveurl.api.
erów.

Dodatkowo, wspomniany dzisiaj pomysł, to zbiór reguł dla meta-resolvera, których resolverów dla którch linków używać najlepiej. Czyli lista preferencji. Oczywiście w meta-resolverze będzie domyślna, a poza tym jak mu jeden sposób zawiedzie (sub-resolver się podda), to będzie brał następny.

Preferencje widzę, poza wpisywaniem w oknie konfiguracji meta-resolvera (co nie jest najwygodniejsze) jako dodatkowe paczki, które mogą być polecane na stronach. Wtedy w meta-resolwerze będzie można wybrać, który zestaw ma obowiązywać (jak user zainstaluje ich pare). 

Czyli np.

script.module.resolverurl-pref.cherry \
↳ meta-resolveurl      (*wymaga*¹) \
↳ resolveurl.jsergio   (*wymaga*¹) \
↳ resolveurl.pl        (*wymaga*¹) \
↳ *plus reguły, od którego z powyższych sub-resolverów zaczynać dla szukanego linku*


#### Podsumowanie

Wygląda na przekombinowane, ale daje spore możliwości. Przy paczkach preferencji się nie upieram ale i one ułatwią userom działanie, a jednocześnie nie zablokują możliwości dostosowania.

Co to daje (albo mi się wydaje, że daje – proszę o krytykę):
* płynna przesiadka (dzięki zależnościom),
* stary kod (bez przeróbek), też powinien zadziałać,
* kod jsergio bez dodatkowego czasu, nawet z numerem wersji (wadą jest ingerencja we źródła, ale to powinien załatwić nowy builder z automatu),
* dowolna ilość nowych resolverów.

Chciałem na koniec zaznaczyć, że wszystkie ogólne zmiany i tak powinniśmy się starać wpychać do jsergio, wtedy więcej osób to ma i to testuje. Ale dzięki podziałowi nie wszystko musi tam trafić a i z pozostałymi zmianami nie musimy się spieszyć (np. z usuwaniem requests).

Mam nadzieję, że choć trochę zrozumiale napisałem.
