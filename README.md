# Rendszerfejlesztés II. gyakorlat

A félév során a CodeMetropolis nevű nyílt forráskódú projekt fejlesztésébe kapcsolódtunk be. A projekt az alábbi linken tekinthető meg: [link](http://codemetropolis.github.io/CodeMetropolis/)

A minden hallgatónak legalább egy issue-t kellett választani, melyen a félév során dolgozott. A feladat az issue teljes életútjának végigkísérése, kezdve a kapcsolódó részek feltérképezésétől, a megvalósításon át, egészen a minőségbiztosítás és dokumentálásig.

Az issuek listája az alábbi linken található: [link](https://github.com/codemetropolis/CodeMetropolis/issues)

Az áltam választott issue: [link](https://github.com/codemetropolis/CodeMetropolis/issues/263)

<b> Mivel ez egy két személy issue, így egy szaktársammal - általában pair programming - történt a fejlesztés. </b>

## A kód
Maga a feautre még nem került mergelésre, így a fejlesztés eredménye még nem látható a CodeMetropolisban. A kódolás az egyetem belső gitlab szerverén történt, így ehhez nem tudok hozzáférést adni. (Nem is vagyok benne biztos, hogy adhatok.) <b>Szükség esetén személyesen meg tudom mutatni a kódot.</b>

## A feladat megismerése

A feladat a játékosnak azon épületek, al-épületek kilistázása, ahol jelenleg áll a parancs kiadásakor. Ezt egy egyszerű command segítségével fogja megtenni (pl: /subbuildings), mely hatására a plugin a placingToRendering fájlból kiolvassa az adatokat és a játékos koordinátáját lekérve megállapítja, mely telken, mely épületen áll, visszaadva az XML hierarchiában szereplő gyerekeit.

A fejlesztés a Plugin.java fájlban fog elkészülni és Spigot szerveren lesz majd használatos, így a módosítandó fájlunk a **sources\integration\spigot-plugin\src\main\java\codemetropolis\integration\spigot\plugin** útvonalon lesz megtalálható.

Hogy a kód szebb legyen, úgy nem kizárólag a Plugin.java fájlban dolgozunk, hanem új classok létrehozásával segítjük munkánkat.

## Megvalósítás
A feladat elkészítése két fő részre oszlik:

### XML feldolgozás
Az első, egy xml fájl feldolgozása, nevezetesena placingToRendering.xml-é, mely a későbbiekben létrejövő városunkban található épületek pontos pozícióját és méretét tartalmazzák. Felépítése hierarchikus, azaz az egyes elemekből és azok gyermekeiből áll. Egy ilyenre példa egy kert (Garden), melyben több épület is lehet, azon lehetnek Floor-ok, Cellar-ok. 
Példa:

``
  <buildable id="da198328-d053-49ad-8ca0-295cd31291db" name="BuildableDepthComparator" type="garden"> 
  
    <position x="39" y="66" z="102"/>    
    <size x="15" y="9" z="17"/>
    <attributes>    
      <attribute name="flower-ratio" value="0.14285714285714285"/>      
    </attributes>    
    <children>    
      <buildable id="0b82b458-68aa-4b8a-9160-9cabbcd7f29b" name="int compare(Buildable b1, Buildable b2)" type="floor">
        <position x="42" y="67" z="105"/>        
        <size x="9" y="13" z="11"/>        
        <attributes>        
          <attribute name="external_character" value="metal"/>          
          <attribute name="character" value="glass"/>          
          <attribute name="torches" value="2"/>          
        </attributes>        
      <children/>      
      </buildable>      
    </children>  
    
  </buildable>
  
``

A fájlt a Java org.w3c.dom interface segítségével dolgoztuk fel. Elsőként csak beégetett pozíciót használtunk, úgy kerestük meg a root Node-ot, mellyel a későbbiekben dolgozni kell. A keresést egy rekurzív metódussal valósítottuk meg, mely addig hívta meg önmagát, míg a megfelelő Node-ot meg nem kaptuk, ezt pedig annak position és size attribútumaival ellenőriztük le. Akkor volt megfelelő az adott Node, ha a mi pozíciónk (még kézzel megadott koordináták) a position és position+ size koordináták közé esik, mind x, mind z tengelyen. Amint kézhez került a feldolgozandó elem, egy másik metódus segítségével lekértük annak children Node-jait.

### A játékos kiszolgálása

A második a játékosnak visszaadni a kívánt sub-building listát. Erre két lehetősége van. Vagy simán használhatja a commandot, vagy plusz lekérést tehet a -d kapcsoló segítségével. Ekkor nem csak az adott Node gyermekei, hanem azok gyermekei, és azok gyermekei stb... kérhetőek le, melyet szintén rekurzív függvény segítségével oldottunk meg. Amennyiben a felhasználó számára több elemnek kell megjelennie a chaten, mint amennyi kifér (default 20 db), úgy a kilistázás oldalakként jelenik meg, melyek között a felhasználó navigálni tud, a parancs után beírva a megfelelő oldalszámot, feltéve, hogy az létezik.

## Felhasználói dokumentáció

Ahhoz, hogy meg tudjuk jeleníteni a munkánkat, a CodeMetropolis által kapott **placintToRendering.xml** fájl elérési útvonalát a plugin által generált mappában (abban az esetben, ha a mappa nem létezik, kézzel kell létrehozni) lévő config.yml fájlban kell elhelyezni a következő módon: **path: C:/Users/valaki/placingToRendering.xml**.

Amint belép a játékos a szerverre, a CodeMetropolis által generált világban találja magát. Könnyen el lehet veszni a felhőkarcolók sokaságában, így segítségre szolgál a **sub-building** parancs.

### Használata

#### Alap funkció

A **t** betű megnyomására megnyílik a chat ablak, melybe a **/sub-building** parancsot kell beírni. Abban az esetben, ha a **config.yml** fájlban nem megfelelően lett megadva a vizsgálni kívánt xml fájlunk útvonala, hibaüzenetet kap a felhasználó. Ellenkező esetben alapméretezetten a jelenlegi pozíción lévő épület gyermekeinek nevét fogja visszaadni a plugin, ezek látszódnak majd a chatben. (amennyiben egy kertben áll, úgy az épületek nevei, egy tér esetén az azon megtalálható kertek, épületek, stb.) Amennyiben ez a lista több, mint 18 elemű, úgy több oldalon keresztül böngészhetjük azokat, a lapozást pedig a parancs után beírt **oldalszám** segítségével tehetjük meg, pl. **/sub-building 3**, megkapva a listánk 3. oldalát - feltéve, hogy az létezik 3. oldal. Természetesen ha olyan oldalt szeretnénk elérni, amely nem található, a játékos hibaüzenetet kap. (Ha nem adunk meg oldalszámot, úgy az első oldalt kapjuk meg alapmréetezetten, de azt elérhetjük ugyanúgy a **sub-building 1** parancs segítségével is

#### Deep search

Felmerülhet a kérdés, mi történik akkor, ha a kilistázott elemeknek további gyermekei vannak a fa hierarchiában. Mi történik, ha a fájlunk valami hasonló felépítésű, mint az alábbi példa?

``

  <building>
  
    <floor>
      <table>
      <chair>
    </floor>
    <ceiling>
      <lamp>
    </ceiling>
    
  </building>


``

Alap módban, ha a **building** területén állunk, csak a **floor** és **ceiling** elemeket jelenítené meg a plugin. Ha mélyebben szeretnénk ez a keresést megtenni, azt a **-d** vagy **-deep** kapcsolóval tehetjük meg. Használata hasonló, a parancs mögé első paraméterként kell beszúrni, majd azt következi opcionálisan az oldal szám: **/sub-building [-d] [oldalszám]**. Itt már érdemes mindig megadni oldalszámot, hiszen feltehetően jóval nagyobb mennyiségű adat lesz, mely nem fog teljes egészében kiférni a chatre.

#### Deep search

Abban az esetben, ha bármely argumentum hibásan van megadva, hibaüzenetet kap a játékos. További segítségért a **/help sub-building** parancsot lehet használni.


## A issue védése során bemutatott prezentáció diái

![image](https://user-images.githubusercontent.com/71877876/173688066-135c5f93-008b-467c-b669-3336b5ebb662.png)

![image](https://user-images.githubusercontent.com/71877876/173688091-e22b7310-19db-4da1-bc3d-2344df072622.png)

![image](https://user-images.githubusercontent.com/71877876/173688113-36318f30-757f-4fd2-a3f8-52a982f9de16.png)

![image](https://user-images.githubusercontent.com/71877876/173688126-f8139a63-766a-451f-8f8a-faa561823ed7.png)

![image](https://user-images.githubusercontent.com/71877876/173688161-e35c7df7-0faa-4775-8c34-017879f73617.png)

![image](https://user-images.githubusercontent.com/71877876/173688175-019dd738-16da-439f-87f2-d8c9990c1b9c.png)

## Minőségbiztosítás

[link](https://github.com/dtomo0420/rf2/blob/main/Min%C5%91s%C3%A9gbiztos%C3%ADt%C3%A1s)

## Dokumentációk

- saját branch és issue karbantartása
- a fejlesztők saját munkanaplójának naprakészen tartása
- heti megbeszélések vezetése
- spigotos issuek egységességének követése
- minőségbiztosítás

## Felhasznált technológiák

- az xml feldolgozása a java org.w3c.dom interface segítségével történt
- a Minecrafthoz és a játékoshoz való kommunikációt a Bukkit segítségével valósítottuk meg

## Összefoglalás, konklúzió

A issueban rögzített feladat teljesítésre került. A projektben a játékosnak egy plusz funkciót is létrehoztunk, melynek segítségével a játékos mélyebben helyezkedő elemeket is kilistázhat. Ekkor a hatékony vizualizáció érdekében érdemes egy oldalszámot is megadni.
