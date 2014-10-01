Az elmúlt hónapokban írtam egy webes alkalmazást, ahol a pénzügyeimet tartom nyilván: Büdzsét készítek vele, feljegyzem a költekezéseimet, stb. A [YNAB](http://www.youneedabudget.com/)-hez hasonló, csak tartalmaz olyan funkciókat, amiket hiányoltam belőle de számomra hasznos lenne (például pénzügyi célok kitűzése és nyilvántartása).

Screenshotot nem tudok mutatni róla, mert jelenleg csak éles adatokkal van feltöltve (értsd: személyes pénzügyi kiadásaim, terveim).

Az alkalmazás egy Single-Page application, a fő üzleti logika a kliensoldalon van lekódolva, a szerver csak adattárolást végez. A kliensoldalon TypeScriptet, KnockoutJS-t és Bootstrap-et használtam.

Az alkalmazás most egy nagyobb refactoron fog átesni, valamint új feature-ök kerülnek bele. Ezeket viszont nem szeretném már TypeScriptben írni.

## Miért szar a TypeScript?
TODO

Nehéz a webre értelmes és/vagy biztonságos programozási nyelvet találni, így a választásom a Kotlin-ra esett.

## Mi a Kotlin?
TODO

## Kihívások
Először is Kotlinban jelenleg még nincs annyi elérhető library, mint JS-hez, így a KnockoutJS kilőve. Számításaim szerint kb. Dom-manipulációt, JQuery-t és néhány JQuery UI hívást tudok végezni.

Mivel csak ilyen primitív eszközök állnak rendelkezésemre, ezért az alkalmazás-logika is hasonló színvonalú lesz: Egy objektum módosításakor a újra felépítem az objektumtól függő felületi elemeket.

Például van egy táblázatom, aminek a `tbody` tag-jére rakok egy ID-t.  
Ezt a táblázatot Kotlinból feltöltöm sorokkal, azokat megformázom, kiszinezem stb. különböző üzleti logika alapján. Valamint tegyük fel, hogy a táblázat egésze függ minden egyes sortól (például a táblázat utolsó sorában megjelenítjök az egyes sorok összegét, vagy a 10. sor színe függ a 9. sor értékétől).  
Majd ha módosul az egyik ilyen sor által reprezentált objektum, akkor kiürítem a fent említett `tbody`-t, és újra lefuttatom az azt feltöltő Kotlin eljárást.  
A megoldás hátránya, hogy lassú lehet, illetve sok felesleges újraszámolást tartalmazhat.

    +----------------+                         +----------------+                   (4) fillTable($('tbody_id'))
    |                |                         |                +---------------------+
    |                |                         |                |                     |
    +-------+--------+                         +----------------+                     |
            |                                                                         |
            | (1) fillTable($('#tbody_id'))            ^ (3) $('tbody_id').empty()    |
            |                                          |                              |
            |                                          |                              |
            |                                          |                              |
            v                                          |                              v
                                                       |
    +----+----------+                          +-----+-+-------+           +----+---------+
    | 12 |    aaaa  |                          | 12  |   aaaa  |           | 12 |   aaaa  |
    | 13 |    bbbb  | +----------------------> | xx  |   xxxx  |           | 14 |   cccc  |
    | 14 |    cccc  |(2) User deletes this row | 14  |   cccc  |           +--------------+
    +---------------+                          +---------------+           | 26 |         |
    | 39 |          |                          | 39  |         |           +----+---------+
    +----+----------+                          +-----+---------+
