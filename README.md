<p align="center">
  <img src="https://engeto.cz/wp-content/uploads/2019/01/engeto-square.png" width="200" height="200">
</p>

# Java 2 - 2. lekce

## Co nás čeká

 - [Input-Output](#input-output)
 
 - [Closeables](#closeables)
 
 - [Streamy](#streamy)
 
## Input-Output

V této části si vysvětlíme základní principy práce se čtením a zápisem dat, někdy zkracované jako I/O. Jedná se o poměrně rozsáhlé téma, které je v Javě řešené na první pohled relativně kompikovaně. Ale ukážeme si, že v moderní Javě je základní použití velice jednodnoduché.

 <p align="left">
  <img src="https://github.com/xthobi/eng_java2_02/blob/main/input_output.png" width="1229" height="178">
</p>

### Typy zdrojů

 - Binární
 
 - Znakové
 
 - Textové s jednoduchou strukturou
 
 - Textové s vnitřní hierarchií (json, XML, ..)

 - Binární soubory jednoduché (nekomprimované média, spustitelné programy, ..)

 - Binární soubory s komplexní vnitřní strukturou (zip, doc, video, ..)

 - Soubory ve filesystému
 
 - Konzole
 
 - Síťová komunikace
 
 - Databáze

Jak vidíme možností je celá řada a naivní přístup načtění celého souboru znak po znaku do jedné proměnné v paměti brzo selže. Je proto potřeba mít možnost definovat různé strategie čtení a zápisu.

### Návrhový vzor Decorator

Pro lepší pochopení Java I/O API je dobré pochopit návrhový vzor Decorator, není to nic složitého. Základním pricnipem je vyhnout se příliš velkému množství specifických tříd lišících se jenom v nějaké dílčí části funkcionality. Lepší řešení je rozdělit funkcionalitu do jednotlivých částí, které jde kombinovat do celku podle potřeby.

Dobrý příklad na pochopení je objednávka kávy, kde se objednávka skládá z několika volitelných komponent:

 - Samotná káva

 - Mléko

 - Sirup s příchutí
 
Řešení je vytvořit jednotný interface a abstraktní třídu Decorator a pak dva druhy tříd:

 - Iniciační třídy, které umožnují vytvořit instanci implementující jednotný interface
 
 - Třídy odvozené od abstraktní třídy Decorator, kde je potřeba do Konstruktoru dané třídy dát již existující třídu
 
Tento pattern pak umožní třídy druhého typu dále řetězit podle potřeby:

```java
import java.util.List;
import java.util.ArrayList;
import java.util.Arrays;

interface Coffee {
    public Double getPrice();
    public List<String> getIngredients();
}

class SimpleCoffee implements Coffee{
    @Override
    public Double getPrice() {
        return 5.0;
    }

    @Override
    public List<String> getIngredients() {
        List<String> list = new ArrayList<>();
        list.add("Coffee");
        return list;
    }
}

abstract class CoffeeDecorator implements Coffee{
    private final Coffee decoratedCoffee;

    // Konstruktor vyžaduje již existující instanci Coffee
    public CoffeeDecorator(Coffee c) {
        this.decoratedCoffee = c;
    }

    @Override
    public Double getPrice() {
        return decoratedCoffee.getPrice();
    }

    @Override
    public List<String> getIngredients() {
        return decoratedCoffee.getIngredients();
    }
}

class WithMilk extends CoffeeDecorator {
    public WithMilk(Coffee c) {
        super(c);
    }

    @Override
    public Double getPrice() {
        return super.getPrice() + 0.5;
    }

    @Override
    public List<String> getIngredients() {
        List<String> list = super.getIngredients();
        list.add("Milk");
        return list;
    }
}

class WithSirup extends CoffeeDecorator {
    public WithSirup(Coffee c) {
        super(c);
    }

    @Override
    public Double getPrice() {
        return super.getPrice() + 1.5;
    }

    @Override
    public List<String> getIngredients() {
        List<String> list = super.getIngredients();
        list.add("Sirup");
        return list;
    }
}

class Main {
  public static void printInfo(Coffee c) {
        System.out.println("Price: " + c.getPrice() + "; Ingredients: " + Arrays.toString(c.getIngredients().toArray()));
    }
  public static void main(String[] args) {
    Coffee c = new SimpleCoffee();
        printInfo(c);

        c = new WithMilk(c);
        printInfo(c);

        c = new WithSirup(c);
        printInfo(c);
  }
}
```

Výstupem je potom:

```console
Price: 5.0; Ingredients: [Coffee]
Price: 5.5; Ingredients: [Coffee, Milk]
Price: 7.0; Ingredients: [Coffee, Milk, Sirup]
```

## Closeables

Jedna z velkých výhod Java je sofistikovaný koncept Garbage Collectoru, který zjednodušuje práci s alokací paměti. Nezvládá ale úplně vše a při práci se zdroji je nejprve potřeba uvolnit zdroje a až pak může Garbage Collector uvolnit paměť.

Všechny zdroje, které je třeba uvolňovat, implementují Interface ```java java.io.Closeable ``` s metodou ```java close()```, jejíž zavolání uvolní zdroje. V nových verzích ale krom tohoto Interface existuje také ještě ```java java.lang.AutoCloseable```, které zajistí při vhodném použití automatické a bezpečné uvolnění zdrojů.

Již jsme se setkali s try blokem, když jsme během cvičení ošetřovali potenciální výjimku. Podobně jde ošetřit i výjimka, která by mohla nastat při práci s se zdroji. To by vypadalo nějak takto:

```java
try {
    // práce se zdroji
} catch (IOException e) {
    // ošetření výjimky
} finally {
    try {
        // uvolnění zdrojů pomocí metody close()
    } catch (IOException e) {
    
    }
}
```

Toto řešení je poněkud těžkopádné, a proto právě vznikla možnost použít AutoClosable spolu s "try with resources". Vzorový zápis by vypadal nějak takto:
```java
try ( //definice zdrojů ) {
    // práce se zdroji
} catch (IOException e) {
    // ošetření výjimky
}
```

Všimněte si, že zde nemáme žádný finally blok a ani žádné volání metody close(), protože zdroje definujeme jako parametry try bloku a proto dojde k jejich automatickému uvolnění.

Při práci se zrdoji je třeba si dát pozor na tyto nejběžnejší chyby:

 - zdroje nejsou ovlňovány vůbec
 
 - zdroje nejsou uvolněny při chybě
 
 - není použit blok <b>try with resources</b> tehdy, když je to možné
 
## Streamy
 
V této kapitole si vysvětlíme základy práce se soubory.

### I/O Streams (I/O Proudy)
 - [<b>Binární streamy</b>](#binární-streamy) - zpracování binárních dat
 - [<b>Znakové streamy</b>](#znakové-streamy) - zpracování znakových dat, včetně automatické konverze znakových sad
 - [<b>Buffered streamy</b>](#buffered-streamy) - využití vyrovnávací paměti pro lepší využití hardwaru
 - [<b>Data streamy</b>](#data-streamy) - třídy pro zpracování primitivních datových typů
 - [<b>Objektové streamy</b>](#objektové-streamy) - zpracování objektů pomocí streamů

### Binární streamy

Pro binární operace použíjeme java.io.FileInputStream a java.io.FileOutputStream.

Pro čtení po byte použijeme metodu int read(). Mohlo by se zdát, že by měl stačit byte, ale je potřeba počítat i hodnotou -1, v případě, že už ve streamu žádná další data nejsou.

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
public class FileStreamExample {
    public static void main(String[] args) throws IOException {
        try (
          FileInputStream in = new FileInputStream("inputFile.txt");
          FileOutputStream out = new FileOutputStream("outFile.txt");
        ) {
            // this have to be int and not byte because it can be -1 or 0-255
            // byte is only -128 to 127
            int c;
            // Read and write one byte at a time
            while ((c = in.read()) != -1) {
                out.write(c);
            }
        }
    }
}
```

### Znakové streamy

Znakové proudy (Character Streams) vycházejí ze tříd java.io.Reader a java.io.Writer. Pro čtění a zápis souborů můžeme použít java.io.FileReader a java.io.FileWriter.

Další znak získáme zavoláním metody int read() na třídě FileReader. Metoda vrátí -1 nebo znak reprezentovaný jedním až čtyřmi bajty v závislosti na použitě znakové sadě.

```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
class Main {
  public static void main(String[] args) throws IOException {
        try (
          FileReader inputStream = new FileReader("inputFile.txt");
          FileWriter outputStream = new FileWriter("outputFile.txt");
        ) {
            int c;
            while ((c = inputStream.read()) != -1) {
                outputStream.write(c);
            }
        }
    }
}
```

#### Znakové sady

Možná si ještě pamatujete, že dříve bylo občas problematické zobrazit správně češtinu. Dnes je naprostá většina kodována pomocí UTF-8. UTF-8 je zajímavý koncept, který kombinuje úsporu místa s možností zapsat libovolné exotické Unicode znaky. Každý znak je namapován na jeden až čtyři bajty. Více detailů na https://en.wikipedia.org/wiki/UTF-8

```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
class Main {
  public static void main(String[] args) throws IOException {
        try (
                FileReader inputStream = new FileReader("inputFile.txt");
                FileWriter outputStream = new FileWriter("outputFile.txt");
        ) {
            int c;
            while ((c = inputStream.read()) != -1) {
                outputStream.write(c);
                String v = String.format("%c",c);
                System.out.println(c +" - "+ v +" - " + caclulateByteSize(c));
            }
        }
    }
    public static int caclulateByteSize(int c){
        int byteSize = 0;
        while (c > 0){
            byteSize++;
            c = c / 256;
        }
        return byteSize;
    }
}
```

```console
97 - a - 1
98 - b - 1
99 - c - 1
10 -
 - 1
283 - ě - 2
353 - š - 2
269 - č - 2
345 - ř - 2
382 - ž - 2
253 - ý - 1
225 - á - 1
367 - ů - 2
10 -
 - 1
22823 - 大 - 2
23478 - 家 - 2
22909 - 好 - 2
```

### Buffered streamy

Další krokem je číst I/O po blocích, v případě souborů je přirozený oddělovač konec řádku. Pro zjednodušení práce s takovými soubory můžeme použít java.io.BufferedReader a java.io.BufferedWriter. BufferedReader má metodu String readLine(), která vrátí celý řádek (bez znaku konce řádku). Pro zápis pak můžeme použít BufferedWriter a metodu write(String text) a pro přidání konce řádku je metoda newLine().

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.lang.Exception;
class Main {
  public static void main(String[] args) {
    try (BufferedReader inputStream = new BufferedReader(new FileReader("inputFile.txt"));
         BufferedWriter outputStream = new BufferedWriter(new FileWriter("outputFile.txt"));) {
            String line;
            while ((line = inputStream.readLine()) != null) {
                outputStream.write(line);
                outputStream.newLine();
            }
        } catch (Exception e){
            e.printStackTrace();
        }
  }
}
```

Ještě jednodušší je použít java.io.PrintWriter, což je obalení java.io.BufferedWriter. Přidáva další metody, které už známe např z System.out - print(String text) a println(String text).

```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.BufferedReader;
import java.io.PrintWriter;
import java.io.IOException;
class Main {
  public static void main(String[] args) {
    try (BufferedReader inputStream = new BufferedReader(new FileReader("inputFile.txt"));
        PrintWriter outputStream = new PrintWriter(new FileWriter("outputFile.txt"));) {
      String line;
      while ((line = inputStream.readLine()) != null) {
        outputStream.println(line);
      }
    } catch (Exception e){
      e.printStackTrace();
    }
  }
}
```

### Data streamy

Data streamy podporují I/O primitivních datových typů (int, long, char, booolean, ..) a Stringů. Všechny datové streamy implementují buď interface DataInput nebo DataOutput a my se podíváme na jejich nejpoužívanější implementace: DataInputStream a DataOutputStream.

 -	<b>double</b> - ```java DataOutputStream.writeDouble()``` ```java DataInputStream.readDouble()```
 -	<b>int</b> - ```java DataOutputStream.writeInt()``` ```java DataInputStream.readInt()```
 -	<b>String</b> - ```java DataOutputStream.writeUTF()``` ```java DataInputStream.readUTF()```

```java
static final String dataFile = "productData";

static final double[] prices = { 25.90, 23.90, 28.90, 12.90 };
static final int[] units = { 28, 14, 36, 4 };
static final String[] descs = {
    "Apple juice",
    "Carrot juice",
    "Orange juice",
    "Tomato juice"
};
```

```java
out = new DataOutputStream(new BufferedOutputStream( new FileOutputStream(dataFile)));

for (int i = 0; i < prices.length; i++) {
    out.writeDouble(prices[i]);
    out.writeInt(units[i]);
    out.writeUTF(descs[i]);
}
```

Metoda writeUTF slouží pro zápis znaků v UTF-8, přičemž je optimalizovaná pro zápis znaků vyskytujících se v západních abecedách, které dovede zapsat úsporně - na jeden bajt.

```java
in = new DataInputStream(newBufferedInputStream(new FileInputStream(dataFile)));

double price;
int unit;
String desc;
double total = 0.0;

try {
    while (true) {
        price = in.readDouble();
        unit = in.readInt();
        desc = in.readUTF();
        System.out.println("Máme na skladě " + unit + " kusů " + desc + " za " + price + "kč/ks");
        total += unit * price;
    }
} catch (EOFException e) {
}
```

Narozdíl od předchozích způsobů načítání dat data streamy neindikují konec souboru návratem nevalidní hodnoty, ale vyhodí výjimku EOFException.

Také je dobré zmínit, že pro každou metodu pro zápis existuje korespondující metoda pro čtení a je na programátorovi, aby je správně použil, protože input stream jsou pouze jednoduchá binární data a nejde z nich poznat, jakého datového typu jednotlivé hodnoty jsou.

### Objektové streamy

Podobně jako datové streamy podporují vstupně-výstupní operace primitivních datových typů, tak objektové streamy podporují zase vstupně-výstupní operace objektů. Většina běžných tříd tuto serializaci podporuje - ty které ji podporují poznáte podle toho, že implementují interface Serializable.

Třídy pro práci s objektovými streamy jsou ObjectInputStream a ObjectOutputStream. Tyto třídy implementují interface ObjectInput, respektive ObjectOutput. Je dobré dodat, že tyto rozhraní rozšiřují nám již známé rozhraní DataInput a DataOutput, což ve výsledku znamená, že objektové streamy nám nabízí krom metod pro práci o objekty i všechny metody pro práci s primitivními datovými typy, které mměly datové streamy.

Zajímavou schopností objektových streamů je také to, že zvládají práci i se zanořenými objekty.

Představte si, že chcete zapsat instanci třídy žák, která by mohla vypadat nějak takto:

```java
public class Student implements Serializable {

    private String name;
    
    private String surname;
    
    private Address address;

}
```

Objektový stream dovede zapsat/načíst nejen studenta, ale i jeho adresu (která je opět objekt).

### Scanner a Tokenizace

Posledním krokem ke čtení je Tokenizace a rozdělení na jednotlivá slova a správná detekce jejich typu.

```java
import java.io.*;
import java.util.Scanner;
class Main {
  public static void main(String[] args) {
        try(Scanner s = new Scanner(new BufferedReader(new FileReader("inputFile.txt")));) {
            while (s.hasNext()) {
                System.out.println(s.next());
            }
        } catch (IOException e){
          e.printStackTrace();
        }
    }
}
```

inputFile.txt

```console
abc 123 123,45 1235.67
ščř 235x
```

Výsledek:

```console
abc
123
123,45
1235.67
ščř
235x
```

```java
import java.io.*;
import java.util.Scanner;
import java.util.Locale;
class Main {
  public static void main(String[] args) {
        double sum = 0;
        try(Scanner s = new Scanner(new BufferedReader(new FileReader("inputFile.txt")));) {
          s.useLocale(Locale.US);
            while (s.hasNext()) {
                if (s.hasNextDouble()) {
                    double v = s.nextDouble();
                    sum += v;
                    System.out.println("Double Value:"+v);
                } else {
                    s.next();
                }
            }
        } catch (IOException e){
          e.printStackTrace();
        }
        System.out.println("DoubleSum: "+sum);
    }
}
```

inputFile.txt

```console
abc 123 1234.56 12346,78
xxx 333 ěšč
```

Výsledek:

```console
Double Value:123.0
Double Value:1234.56
Double Value:333.0
DoubleSum: 1690.56
```

## Práce se soubory

V této sekci si ukážeme základy práce se soubory pomocí ```java java.nio.file```.

### Filesystem a více platforem

Jak jsme si už dříve vysvětlovali, Java je vytvořená s přístupem pro psaní a spouštení stejného kódu na více platformách. A jeden z velkých rozdílů mezi Windows and Linuxem je práce se souborovým systémem.

Na Windows je každá jednotka označená písmenem a složky jsou oddělené zpětným lomítkem \. Základní attributy jsou sdílené pro všechny uživatele stejně a práva k souborům je možné nastavit relativně komplikovaně (minimálně na NTFS).

```console
C:\Users\Abc\file.txt
```

Na Linuxu je jenom jeden kořen / a všechno je namapované jako adresář. Práva se nastavují ve třech skupinách (vlastník, skupina, všichni) a je možné nastavit 3 druhy práv nezávisle(čtení, zápis, spuštění).

```console
/home/abc/file.txt
```

Navrhnout jednotné API, které by umožnilo pracovat s oběma systémy bez kompatibilně není nic jednoduchého. Proto první pokus v Javě narazil na jisté limity a Java NIO přišlo s robustnějším řešením.

### Path

První třídu, kterou si popíšeme je Path. Slouží k reprezentaci cesty k adresáři nebo souboru.

Absolutní cesta - Cesta k souboru od výchozího místa filesystému

```console
C:\Users\Abc\file.txt
````

Relativní cesta - Cesta k souboru od nějakého jiné cesty ve filesystému

A: C:\Users\Abc\ B: C:\Users\Abc\Xyz\file.txt

A -> B .\Xyz\file.txt B -> A ..\

#### Vytvoření

K vytvoření instance Path, zavoláme metodu get na pomocné třídě Paths.

```java
Path p1 = Paths.get("/tmp/foo");
Path p2 = Paths.get(args[0]);
Path p3 = Paths.get(URI.create("file:///Users/joe/FileTest.java"));
Path p4 = FileSystems.getDefault().getPath("/users/sally");
Path p5 = Paths.get(System.getProperty("user.home"),"logs", "foo.log");
```

#### Normalizace

Cestu můžeme definovat pomocí . (aktualní adresář) a .. (nadřazený adresář). Často se ale hodí vypočítat cestu, která reprezentuje nejjednodušší cestu k danému místu. To docílíme zavoláním metody normalize()

C:\Users\Abc\..\.\.\Abc\file.txt -> C:\Users\Abc\file.txt

#### Kontrola existence

Instanci Path jde vytvořit nezávisle jestli daný soubor existuje nebo ne. Což je přirozené, jak jinak by bylo možné vytvořit nový soubor? Ne vždy je ale potřeba vytvořit nový soubor, možných operací se soubory je celá řada.

Můžeme použít metodu exists(Path, LinkOption...) nebo notExists(Path, LinkOption...). Při kontrole existence cesty může nastat jedna ze tří situací:

Soubor existuje a podařilo se ověřit existenci
Podařilo se verifikovat, že soubor neexistuje
Aplikace nemohla zjistit stav souboru. Důvodů je celá řada, např. aplikace nemá práva k dané části filesystému.

#### Kontrola práv

Files.isReadable(Path) -> kontrola jestli aplikace má práva ke čtení na dané cestě Files.isWritable(Path) -> kontrola jestli aplikace má práva ke zápisu na dané cestě Files.isExecutable(Path) -> kontrola jestli aplikace má práva ke spuštení na dané cestě

#### Smazání

Files.delete(path) -> Smaže danou cestu (soubor nebo prázdnou složku), ale pokud cesta neexistuje, tak vyhodí výjimku NoSuchFileException Files.deleteIfExists(Path) -> Smaže danou cestu

```java
try {
    Files.delete(path);
} catch (NoSuchFileException x) {
    System.err.format("%s: no such" + " file or directory%n", path);
} catch (DirectoryNotEmptyException x) {
    System.err.format("%s not empty%n", path);
} catch (IOException x) {
    // File permission problems are caught here.
}
```

#### Kopírování
```java
Files.copy(Path, Path, CopyOption...)
```

REPLACE_EXISTING - Nahradí neprázdnou složku COPY_ATTRIBUTES - Překopírování atributů NOFOLLOW_LINKS - Bez použití symbolických linků

#### Přesun
```java
Files.move(Path, Path, CopyOption...)
```

REPLACE_EXISTING - Nahradí neprázdnou složku ATOMIC_MOVE - Provede přesun jako atomickou opraci, umožní ostatním přečíst soubor na novém místě až v momentě, kdy bude přesunutý kompletní obsah

#### Výpis souborů ve složce

```java
Path dir = ...;
try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir)) {
    for (Path file: stream) {
        System.out.println(file.getFileName());
    }
} catch (IOException | DirectoryIteratorException x) {
    // IOException can never be thrown by the iteration.
    // In this snippet, it can only be thrown by newDirectoryStream.
    System.err.println(x);
}
```
  
#### Vytvoření nové složky

```java
Path dir = ...;
Files.createDirectory(path);
```
  
### Čtení a zápis menších souborů
  
Nejjednodušší způsob čtení a zápisu souborů je pomocí pomocné třídy Files.

Čtení:

```java
Path file = ...;
byte[] fileByteArray = Files.readAllBytes(file);
Path file = ...;
Charset charset = StandardCharsets.UTF_8;
List<String> fileLines = Files.readAllLines(file, charset);
```

Zápis:

```java
Path file = ...;
byte[] = ...;
Fileswrite(path, byte[]);
Path file = ...;
Charset charset = StandardCharsets.UTF_8;
Iterable< extends CharSequence> fileLines = ...;
write(file, fileLines, charset);
```
