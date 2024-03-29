# Tesztelés

## Unit tesztelés

### Felelős: @h982596 

### Indoklás:
Teszteléshez a Unit teszteket választjuk, mivel ezzel a vizsgált metódusok helyességét tudjuk megállapítani, jelen esetben, hogy a feldolgozott XML fájlban jó értékeket kapunk-e vissza, megfelelő módon keresi ki az XML hierarchiában a keresett elemeket.

### Tesztek megvalósítása:

A tesztfájlok a `sources\integration\spigot-plugin\src\test\java` útvonalon lettek létrehozva. Ezek azért voltak szükségese számunkra, hogy az elkészített feature megfelelő működését tudjuk mérni.

A tesztelések során két metódust vizsgáltunk, ezek a `findNode` és a `getNodes`. Mivel egyik metódus sem ad vissza semmilyen értéket (nincs return value), és általában a tesztek során az `assert` metódusokkal valamilyen visszaadott érték vizsgálata történik, így picit át kellett alakítani a `SubBuildingCommand` osztályt annak érdekében, hogy megfelelően lehessen a metódusokat tesztelni. Ehhez getterekre és setterekre volt szükségünk, illetve egy új `init` metódus, mely az XML fájl beolvasásáért felel még azelőtt, hogy a játékos kiadná a parancsot, és meghívódna az `onCommand` függvény. Ennek oka, hogy szükség lett volna egy `Player` objektumra, akivel meghívjuk a metódust (paraméterként átadva), viszont ezt se mockolással, se más módszerrel nem lehet megtenni, így az XML feldolgozását ki kellett szervezni máshova.

Amint ezek megfelelően megtörténtek, elkészültek a teszt metódusok a teszt osztályban. Az XML útvonala ebben az esetben be lett égetve, de természetesen dinamikusan is lehetne változtatni.

Először a fentebb említett `findNode` metódus lett tesztelve. Minden esetben inicializálunk egy SubBuildingCommand objektumot, majd meghívjuk rá a metódust. Ahogy említve volt, ez nem ad vissza semmit, hanem az osztály egy adattagját, nevezetesen a `buildingList`-et tölti fel értékekkel, így annak kiolvasására lesz sükség. Lekérjük a megfelelő `Node`-t, kasztoljuk `Element`-re, és megvizsgáljuk a `name` attribútumát. Ha megfelelő koordinátához az `assertEquals` metódusban megfelelő stringet adunk meg, a teszteset hibátlanul lefut. 

Öt különböző elemre lett futtatva:

```java
@Test
    void testFindNode_1() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 3, 3);
        Node node = subBuildingCommand.getBuildingList().get(0);
        Element element = (Element) node;
        assertEquals("codemetropolis", element.getAttribute("name"));
    }

    @Test
    void testFindNode_2() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 199, 62);
        Node node = subBuildingCommand.getBuildingList().get(0);
        Element element = (Element) node;
        assertEquals("CdfTree createElements()", element.getAttribute("name"));
    }

    @Test
    void testFindNode_3() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 76, 15);
        Node node = subBuildingCommand.getBuildingList().get(0);
        Element element = (Element) node;
        assertEquals("BuildableTree", element.getAttribute("name"));
    }

    @Test
    void testFindNode_4() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 112, 70);
        Node node = subBuildingCommand.getBuildingList().get(0);
        Element element = (Element) node;
        assertEquals("x", element.getAttribute("name"));
    }

    @Test
    void testFindNode_5() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 12, 187);
        Node node = subBuildingCommand.getBuildingList().get(0);
        Element element = (Element) node;
        assertEquals("executor", element.getAttribute("name"));
    }
```

#### A futás eredménye pedig:

![image](https://user-images.githubusercontent.com/71877876/173691338-e7142d07-2c0b-4979-99c6-02379565d214.png)

Ezt követően a `getNodes` metódus tesztelése következett. Ennek lényege, hogy az előbbi, `findNote` által megtalált elemnek lekéri, majd eltárolja a gyermekeit. A metódus harmadik paramétere egy boolean érték, mely azért felel, hogy "mély" keresést végez, ha szeretnénk. Ha igen, azaz `true` értéket adunk meg, úgy nem csak az adott `Node` közvetlen gyermekeit kapjuk meg, hanem azok gyermekeit is, és így tovább rekurzívan minden elemet összegyűjtünk. 

Először simán, közvetlen gyermekekre készültek el a tesztesetek. Vizsgálva volt olyan, ahol nem kellett semmit visszaadnia (azaz az adott elemnek nincs leszármazottja), volt, ahol 1, illetve több gyermek található meg.

```java
@Test
    void testGetNodesWithShallowSearchWithNoChild() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 36, 190);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, false);
        assertEquals(0, subBuildingCommand.getShallowBuildingList().size());
    }

    @Test
    void testGetNodesWithShallowSearchWithOneChildren() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 36, 163);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, false);
        assertEquals(1, subBuildingCommand.getShallowBuildingList().size());
    }

    @Test
    void testGetNodesWithShallowSearchWithTwoChildren() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 12, 187);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, false);
        assertEquals(2, subBuildingCommand.getShallowBuildingList().size());
    }

    @Test
    void testGetNodesWithShallowSearchWithThreeChildren() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 172, 62);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, false);
        assertEquals(3, subBuildingCommand.getShallowBuildingList().size());
    }

    @Test
    void testGetNodesWithShallowSearchWithMoreChildren_1() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 169, 59);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, false);
        assertEquals(11, subBuildingCommand.getShallowBuildingList().size());
    }

    @Test
    void testGetNodesWithShallowSearchWithMoreChildren_2() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 143, 62);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, false);
        assertEquals(6, subBuildingCommand.getShallowBuildingList().size());
    }

    @Test
    void testGetNodesWithShallowSearchWithMoreChildren_3() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 15, 15);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, false);
        assertEquals(67, subBuildingCommand.getShallowBuildingList().size());
    }
```

#### Eredmény:

![image](https://user-images.githubusercontent.com/71877876/173691305-19e6c8a1-c991-4e87-ab4f-ea06c66a3a8d.png)

Az utolsó körben az **extra** mély keresés eredményessége lett tesztelve, hasonló módon, mint a sima. Kerestünk 0 gyermekes elemet, illetve gazdagabb családdal rendelkezőt is.

```java
@Test
    void testGetNodesWithDeepSearchWithNoChild() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 18, 193);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, true);
        assertEquals(0, subBuildingCommand.getShallowBuildingList().size());
    }

    @Test
    void testGetNodesWithDeepSearchWithOneChildren() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 18, 102);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, true);
        assertEquals(1, subBuildingCommand.getDeepBuildingList().size());
    }

    @Test
    void testGetNodesWithDeepSearchWithMoreChildren_1() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 79, 70);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, true);
        assertEquals(6, subBuildingCommand.getDeepBuildingList().size());
    }

    @Test
    void testGetNodesWithDeepSearchWithMoreChildren_2() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 76, 67);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, true);
        assertEquals(14, subBuildingCommand.getDeepBuildingList().size());
    }

    @Test
    void testGetNodesWithDeepSearchWithMoreChildren_3() {
        SubBuildingCommand subBuildingCommand = new SubBuildingCommand(path);
        subBuildingCommand.init();
        subBuildingCommand.findNode(subBuildingCommand.getRoot(), 137, 12);
        subBuildingCommand.getNodes(subBuildingCommand.getBuildingList().get(0), 0, true);
        assertEquals(61, subBuildingCommand.getDeepBuildingList().size());
    }
```

#### Eredmény:

![image](https://user-images.githubusercontent.com/71877876/173691264-2e3e1f48-1a4f-49a1-9323-03a44e81b26d.png)

#### Végeredmény: 

Minden elkészített teszteset eredményesen lefutott, így kijelenthető, hogy megfelelően működik az elkészült feature.

![image](https://user-images.githubusercontent.com/71877876/173691244-7921adff-1c89-4747-9dc0-9fac0a7ac644.png)

## Végfelhasználói tesztek

### Felelős: @h982508 

#### Indoklás:
A feladatunk az volt, hogy a világban mászkáló játékosnak, amennyiben az általunk megírt parancsot beírja, majd futtatja a chatben, az adott épületen ahol áll, gyermekeit megjelenítsük. Ahhoz, hogy igazoljuk az élkészített funkció helyességét, be fogunk lépni a világba és a játékos perspektívájából fogjuk megvizsgálni.

#### Előkészület:
- [sourcemeter](https://www.sourcemeter.com/) telepítése
- [minecraft](https://www.minecraft.net/en-us) telepítése
- [codemetropolis](http://codemetropolis.github.io/CodeMetropolis/) telepítése
- megfelelő Java verzió 
- [Spigot szerver](https://getbukkit.org/download/spigot) telepítése

#### Megvalósítás:
- [minecraft világ legenerálása az alábbiak szerint](https://okt.sed.hu/rf2/gyakorlat/projekt/build-kabinet/)
- amint ez megtörtént, a legenerált világot, mely a distro mappába készül el, másoljuk be a szerver mappájába (felülírva a jelenlegi `world` mappát), illetve a `spigot-plugin-1.4.0.jar`-t a szerver Plugins mappájába
- indítsuk el a szervert, majd amit sikeres betöltött, megnyitva a Minecraftot, kattintsunk a multiplayer gombra
- keressük meg az `Add server` gombot, írjuk be az ip cím helyére, hogy `localhost`
- amint elérhető lesz a szerver, lépjünk fel rá, és használhatjuk is a plugint. Ehhez az alábbi linken találunk [segítség](http://gitlab-okt.sed.hu/rf2-2022/CodeMetropolis-h16/issues/8)et

#### Próba:
  - keressünk a `placintToRendering.xml`-ben egy tetszőleges elemet, legyen ez pl. a `IFLPreferencePage`. Ez az XML-ben az alábbiak szerint néz ki:
![image](https://user-images.githubusercontent.com/71877876/173691146-a625a9a3-4cdb-42d4-8023-a9c1dcf28a9b.png)
  - navigáljunk el az adott koordinátára az XML alapján, melyek a következőek lesznek: `x="24" y="68" z="244"`
  - írjuk be a parancsot, majd a következő képet kell, hogy kapjuk:
 ![image](https://user-images.githubusercontent.com/71877876/173691183-bcd3799a-c0af-4333-adff-a38788ebb2d6.png)
  - ahogy az látszik, nem kaptunk meg minden elemet, amit az XML-ben láthattunk. Ennek oka, hogy több elem van, mint amennyit a játék meg tud jeleníteni egyszerre. Erre a célre lett **extra funkcióként** kifejlesztve, hogy oldalakra bontva jelenítse meg a plugin az adatokat
  - annak érdekében, hogy a maradék adatot is elérjük, "lapozzunk", írjunk a parancs után egy tetszőleges számot. Ebben az esetben, ahogy a képen is látszik, csak 2 oldal van, így próbáljuk meg elérni azt:
![2022-05-01_13.00.45](/uploads/ccd08460424c85d18cb612d8d35f1ea5/2022-05-01_13.00.45.png)
  - meg is kaptuk a lekérni kívánt adatokat

##### + Extra funkció:
  - észrevehetjük az XML-ben, hogy vannak elemek, melyeknek további gyermekei vannak:
![image](https://user-images.githubusercontent.com/71877876/173690979-ce901099-4aca-4748-98b0-19ad0e856d9f.png)
  - ezek eléréshez a játékon belül, a parancsot használjuk a `-d`, vagy `-deep` kapcsolóval, az eredmény a következő lesz:
![image](https://user-images.githubusercontent.com/71877876/173690935-b061d8ce-90b7-4492-8e69-8cd72ae385a9.png)
  - ahogy a fájlban is láthattuk a gyermekeket, úgy a játékban is megjelennek a kapcsoló hatására

##### Hibás használat

Mi történik akkor, ha rosszul használjuk a programot? Természetesen ezek is le lettek kezelve. 

Olyan oldal megadása, ami nem létezik:

![image](https://user-images.githubusercontent.com/71877876/173690828-36340a9a-0d22-4a51-8d4a-c7d60ddc6c60.png)

Hibás paraméter megadása:

![image](https://user-images.githubusercontent.com/71877876/173690872-51f1e6fa-5706-49e2-b649-9cc2b6736298.png)

# Szoftverelemzés: minőségriport kkódelemzés alapján

### Felelős: @h982508 és @h982596 

### Indoklás:
Ahhoz, hogy számszerű adatokat kaphassunk az elkészült funkcióról, minőségriportra lesz szükségünk. Ezzel kiválasztott mertikákat nyerhetünk ki, az esetleges hibákat, bad smelleket vehetünk észre, egy szóval számunkra fontos adatok birtokába kerülhetünk.

## Minőségriport összefoglaló dokumentum
A codemetropolis egy szoftveres vizualizációs eszköz, amely a forráskód felhasználásával és különböző metrikák támogatásával egy minecraft világot tud létrehozni. Segítségével megjeleníthető a program belső szerkezete, illetve a különböző forráskód elemek is. Ezen szoftverkomponensek egy generált városrészként vannak ábrázolva, például az osztályokat épületek, a metódusokat emeletek szimbolizálják. Ezen vizualizáció hatékony megjelenítést biztosít a kisméretű projektek esetén, azonban nagyobb programoknál már átláthatatlan metropoliszok generálódnak. Ezen zűrzavar megoldására nyújt megoldást az általunk megvalósított feature, amely a jelenlegi pozíción lévő épület gyermekeinek nevét tudja lekérdezni (továbbiakban: sub-building-ek).

### Funkcionalitás
A feature az elvárt követelményeknek megfelelően működik, hibamentesen lefordul. A helyes működésről unit, illetve végfelhasználói tesztekkel győződtünk meg.

- **végfelhasználói tesztek:** megvizsgáltuk mindhárom megvalósított parancsot, valamint a parancs helytelen használatát is.

- **unit tesztek:**

![image](https://user-images.githubusercontent.com/71877876/173691494-deb652e5-5ead-4351-b647-1b5d57c1609e.png)

### Felhasználói élmény
Magának a programnak elsődleges célja a forráskód hatékony vizualiációja. A megvalósított feature jelentősen növeli a program ezen funnkcióját. Több lehetőséget is biztosítunk a felhasználónak a sub-building-ek listázására:

- **/sub-building parancs megadása:** a jelenlegi pozíción lévő épület gyermekeinek nevét tudja lekérdezni. Hátránya: nagy méretű metropoliszok esetén ez nem jelent megoldást, mivel sok esetben egy pozición több gyermek objektum van, mint amennyit mi egy oldalon meg tudnánk jeleníteni. 

- **/sub-building [oldalszám] parancs megadása:** működése megegyezik a korábbi paranccsal, azonban ezesetben egy oldalszám megadásával lehetőség van több oldalon keresztül is listázni az eredményeket.

- **/sub-building -d [oldalszám] parancs megadása:** a korábbi funkciók a jelenlegi pozición lévő épület közvetlen gyerekeinek listázásra voltak alkalmasak. Ez sok esetben azonban nem nyújt kellő információt a felhasználónak, így egy plusz funkciót is létrehoztunk. A -d kapcsolóval nem csupán a gyerekek, hanem a gyerekek gyerekeinek listását is biztosítja.

### Teljesítmény
A megvalósított feature számításigénye igen alacsony, így a parancs kiadása után - akár nagyméretű metropolisz esetén is - a felhasználó a másodperc töredéke alatt a kívánt információkhoz juthat.

### Kompatibilitás
A CodeMetropolis használatához a következő programok telepítése szükséges:

- JDK 8, a parancssori eszközök futtatásához

- Minecraft 1.8 (vagy frissebb verzió), a minecraft világ megjelenítéséhez

- SourceMeter, a forráskód metrikáit tartalmazó gráffájl előállításához

### Hordozhatóság
A program működéséhez a kompatibilitás pontban meghatározott környezet előállítása szükséges. Amennyiben ezek biztosítottak a programnak működni kell.

### Megbízhatóság
A feature megvalósítása során felkészültünk a külöböző hibák kezelésére. 

- Amennyiben a feldolgozandó xml útvonala nem megfelelően lett megadva a felhasznált hibaüzenettel értesítjük erről.

- a /sub-building [oldalszám] parancs megadása során a felhasználó tetszőleges számot beírhat. Ez túlindexeléshez vezethet, ha nem létezik az adott oldal. Hibaüzenet formájában értésítjük erről a felhasználót.

- Abban az esetben, ha valamely argumentum hibásan van megadva a játékos hibaüzenetet kap.

- Lehetséges, hogy a felhasználó önhibáján kívül nem ismeri a listázás használatát, ekkor a /help sub-building parancsot tudja segítségül hívni.

### Biztonság és biztonságosság
A biztonságosság kérdése az általunk megvalósított feuture-ben nem releváns kérdés, mivel esetünkben nem beszélhetünk autentikációról, vagy mondjuk titkosított csatornán áramló adatokról.
Biztonság szempontjából az általunk megvalósított osztályok biztonságos értékélést kaptak, így a kód biztonságos.

### Karbantarthatóság
Karbantarthatóság szemléltetése esetén egy olyan metrikát kell választanunk, amely képes a összetettségének reprezentálására. Mi a cyclomatic complaxity-t választottuk. Az elemzés eredménye:

| Osztály | Értéke |
| --- | ----------- |
| **Plugin.java** | 1 |
| **FileManager.java** | 4 | 
| **SubBuildingCommand.java** | 73 | 

A táblázatból kiolvasható, hogy a SubBuildingCommand.java osztály ciklomatikus komplexitása jóval nagyobb, mint a másik két osztálynak. Ennek oka, hogy a feature lényegi megvalósítása itt történt. A magas érték ellenére, ez mégis elfogadható mivel betartottuk az OOP elveket és törekedtünk a kód olvashatóságának maximalizálására. A továbbiakban a feature bővítése nem jelenthet problémát, ezt a kód teljeskörű dokumentáltsága is elősegíti.

## Kód auditálás statikus elemzési módszerrel, PMD rendszer segítségével
A PMD alkalmazás 2 darab blokkoló hibát és 21 sürgető hibát detektált, kritikus hiba nem fordult elő.

Az elemzése riportja: [pmd-report.txt](/uploads/cea810776a533e1d121b2ca390f709b6/pmd-report.txt)

### Blokkoló hibák
- **LocalVariableNamingConventions**: Néhány változó elnevezése nem felel meg a javaban megszokott nevezéktannak. *Javítása szükségeses? A program futását nem akadályozó hiba, javítása nem sürgős.*
- **VariableNamingConventions**: A fent említett LocalVariableNamingConventions esettel egyezik meg. *Javítása szükségeses? A program futását nem akadályozó hiba, javítása nem sürgős.*

### Sürgető hibák
- **BeanMembersShouldSerialize**: ha egy osztály egy bean, vagy egy bean hivatkozik rá közvetlenül vagy közvetve, akkor szerializáltnak kell lennie. *Javítása szükségeses? Nem okoz hibát, de érdemes javítani.*
- **MethodArgumentCouldBeFinal**: azon metódus argumentumok, melyek a metóduson belül nem kerülnek megváltoztatása final-é nyilváníthatóak. *Javítása szükségeses? Javítása szükségtelen.*
- **CommentRequired**: bizony elemeknél a javadoc hiányát jelzi. *Javítása szükségeses? Ahol szükségesnek gondoltuk rögzítettünk javadoc-ot. További dokumentáció elhelyezése a forráskódban már az olvashatóság rovására menne, így nem szükséges további javadoc elhelyezése.*
- **EmptyStatementNotInLoop**: üres utasítás, vagy egy pontosvessző önmagában. A legtöbb esetben ez a dupla pontosvessző esete. *Javítása szükségeses? Ezen problémák javítása javasolt.*
- **ImmutableField**: azonosítja azon objektumokat, melyek értéke az inicializálást követően nem változik. Ezen adattagok láthatóságát érdemes private-ra módosítani. *Javítása szükségeses? Javítása szükségtelen.*
- **SingularField**: azon változókat azonosítja, melyeket globális változóként definiálunk, viszont csak egy alkalommal lokálisan hazsnálunk. Érdemes őket lokális változóként létrehozni. *Javítása szükségeses? Javítása szükségtelen.*
- **AvoidPrintStackTrace**: printStackTrace helyett logger hívás használatát javasolja. *Javítása szükségeses? Javítása szükségtelen.*
- **AvoidDuplicateLiterals**: duplikált String literálokat tartalmazó kód helyett állandó String mezőként való deklarálását javasol. *Javítása szükségeses? Nem súlyos hiba, javítása szükségtelen és nem is eredményezne lényegi változást.*
- **AtLeastOneConstructor**: a nem statikus osztályok esetén elvárás legalább egy konstruktor. *Javítása szükségeses? Feleslegesen "szemetelnénk" tele  kódot. A program későbbi változtatása esetén sem jelentene problémát ezen kostruktorok létrehozása, mivel a program teljeskörüen dokumentálva lett.*
- **CommentSize**: a kódban elhelyezett commentek méretei meghaladják az előre meghatározott határt. *Javítása szükségeses? Minden esetben próbáltuk a legegyszerűbb megfogalmazni a commenteket, ha tovább egyszerűsítenénk őket, az a dokumentálás minőségének romlását eredményezné. Javítása nem javasolt.*
- **LawOfDemeter**: Demeter törvénye alapján "csak barátokkal beszéljünk". Segítségével csökkenthető az osztályok, illetve objektumok közötti kapcsolatok száma. *Javítása szükségeses? Zavaró, de nem veszélyes hiba. Javítása nem fontos.*
- **CyclomaticComplexity**: egy metrika, amely a forráskódban található elágazosákból felépülő gráf pontjai, és a köztük lévő élek alapján számítható. *Javítása szükségeses? Értéke 8+ felett magasnak számít, ezek javítása javasolt.*
- **CognitiveComplexity**: a nehezen értelmezhető viselkedést mérő metrika. Használata olvasható, érthető kódot eredményez. *Javítása szükségeses? A 15+ feletti értékek esetén szükséges ezen hibákat javítani. Nálunk nem fordul elő ilyen magas kognitív koplexitás.*
- **AvoidInstantiatingObjectsInLoops**: cikluson belüli objektumok létrehozásának lehetőségét, illetve felhasználhatóságát ellenőrzi. *Javítása szükségeses? Mindenképp érdemes megvizsgálni ezen problémákat, szükség esetén javításuk szükséges.*
- **AvoidLiteralsInIfCondition**: a hard-coded változók használata helyett, javasolt statikus változók felvétele az if ciklusokban. *Javítása szükségeses? Javításuk szebb kódot eredményez.*
- **LongVariable**: túl hosszú a deklarált változónk neve. *Javítása szükségeses? Javításukkal a változók nem lennének kellően "beszédesek".*
- **ShortVariable**: túl rövid a deklarált  változónk neve. *Javítása szükségeses? Bizonyos esetekben - mondjuk koordináták esetén - az x/z/y nevezéktan egyszerűsége ellenére megfelelően reprezentálja a jelölt adatot, így változtatásuk feleslegesen bonyolítaná a kódot.*
- **LocalVariableCouldBeFinal**: azon lokális változók, melyeket az inicializálást követően nem változtatunk meg final-é tehetők. *Javítása szükségeses? Javítása a kód minőségén javít.*
- **GodClass**: egy metrika, amely az "isten osztályok" megjelenését célzott megelőzni. Azon osztályok tartoznak ide, melyek túl sok mindent csinálnka, nagyon nagyon és túl bonyolultak. Az objektumorientált programozás mérésére hatákonyan alkalmazható. *Javítása szükségeses? Esetünkben a kód további egységeinek kiszervezése nehezen értelmezhető kódhoz vezetne.*
- **LiteralsFirstInComparisons**: a stringek összehasonlítása esetén a literálokat érdemes előre helyezni, ha a második argumentum nulla. *Javítása szükségeses? Javítása a kód minőségén javít.*
- **LinguisticNaming**: ellenőrzi a nyelvi elnevezési antimintákat. A megnevezett mezőket ellenőrzi mintha logikai értékeket adnának vissza. Továbbá vizsgálja, hogy a getterek és a "to"-val kezdődő metósudok visszaadnak-e valamit. *Javítása szükségeses? Nem fontos, csupán zavaró hibák. Javításuk nem fontos.*

## Metrikák
Ebben a fejezetben a codemetropolis codemetropolis.integration.spigot.plugin package-ben lévő osztályok metrikáit dokumentálom a megvalósítás előtt, illetve után.

### A codemetropolis.integration.spigot.plugin package metrikái a megvalósítás előtt:

| Metrika | Értéke |
| --- | ----------- |
| **LLOC**(Logical lines of code) | 1 |
| **WMC**(Weighted method count) | 0 | 
| **Complexitiy** | low | 
| **Size** | low | 

### A Plugin osztály metrikái a megvalósítás előtt:

| Metrika | Értéke |
| --- | ----------- |
| **LLOC**(Logical lines of code) | 1 |
| **WMC**(Weighted method count) | 0 | 
| **NOM**(Number of methods) | 0 | 
| **Complexitiy** | low | 
| **Size** | low-medium | 

### A továbbiakban a megvalósítást követő metrikákat szemléltetem

### A codemetropolis.integration.spigot.plugin package metrikái a megvalósítást követően:

| Metrika | Értéke |
| --- | ----------- |
| **LLOC**(Logical lines of code) | 176|
| **WMC**(Weighted method count) | 66 | 
| **Complexitiy** | low | 
| **Size** | low | 

### A codemetropolis.integration.spigot.plugin package a megvalósítást követően már három osztályt tartalmaz:
- SubBuildingCommand,
- FileManager,
- Plugin
a továbbiakban ezen osztályok metrikáit szemléltetem.

### A SubBuildingCommand osztály metrikái:

| Metrika | Értéke |
| --- | ----------- |
| **LLOC**(Logical lines of code) | 157|
| **WMC**(Weighted method count) | 61 | 
| **NOM**(Number of methods) | 24 | 
| **Complexitiy** | medium-height | 
| **Size** | low-medium | 

### A FileManager osztály metrikái:

| Metrika | Értéke |
| --- | ----------- |
| **LLOC**(Logical lines of code) | 13 |
| **WMC**(Weighted method count) | 4 | 
| **NOM**(Number of methods) | 2 | 
| **Complexitiy** | low | 
| **Size** | low | 

### A Plugin osztály metrikái:

| Metrika | Értéke |
| --- | ----------- |
| **LLOC**(Logical lines of code) | 6 |
| **WMC**(Weighted method count) | 1 | 
| **NOM**(Number of methods) | 1 | 
| **Complexitiy** | low | 
| **Size** | low-medium | 

## Kód auditálás statikus elemzési módszerrel, Checkstyle rendszer segítségével
 Az elemzéshez a Google Checks alapbeállításait használtuk.

### Az elemzés eredménye a feature megvalósítása előtt

A Plugin.java fájl 5.sorában a hiányzó javadoc-ra figyelmeztet a rendszer.

### Az elemzés eredménye a feature megvalósítása után

- Name 'X' must match pattern 'X'

- 'X' is not preceded with whitespace

- 'typecast' is not followed by whitespace

- First sentence of Javadoc is missing an ending period

- Line is longer than X characters (found X)

- Each variable declaration must be in its own statement

- 'X' child has incorrect indentation level X, expected level should be X

- 'X' has incorrect indentation level X, expected level should be X

- 'X' is not followed by whitespace

- Block comment has incorrect indentation level X, expected is X, indentation should be the same level as line X

- Javadoc tag 'X' should be preceded with an empty line

- Line contains in import group before 'X'

- Missing a Javadoc comment

- Comment has incorrect indentation level X, expected is X, indentation should be the same level as line X

### Az elemzés eredménye a feature megvalósítása után, kördiagrammon ábrázolva a szabálysértések arányának megfelelően

![image](https://user-images.githubusercontent.com/71877876/173690730-77c437a3-e15b-4ec6-9233-29578020fad5.png)

