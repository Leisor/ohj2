Kommentti: Ottaisin Jonnen kommentit huomioiden edelleen `instanceof`:n jo perinnässä ([PR 36](https://github.com/ohj-perus-jy/ohj2/pull/36). En niinkään `if`:ien kanssa kapseloinnin rikkomiseen — muuten kuin juuri varoittavana esimerkkinä — vaan ihan puhtaasti havainnollistamaan perintää ja mitä se tarkoittaa tyyppihierarkiassa: alityypin ilmentymät eli oliot ovat aina myös ylityypin ilmentymiä. Mitä isommin tästä puhuu, sitä parempi muotoilla lisämateriaaliksi.


<details closed><summary><b>Ajonaikainen tyypintarkistus</b></summary>

(drafti)

Voimme havainnollistaa ja tarkistaa, miten perintäsuhteet toimivat Java:n `instanceof`-operaattorin avulla. Nimensä mukaisesti operaattori kertoo, onko olio tietyn luokan tai tyypin ilmentymä palauttaen totuusarvon `true` tai `false`. Tämän hyöty on, että ohjelmakoodissa voi ajonaikaisesti tarkistaa mitä tyyppiä muuttuja on tarkemmin kuin muuttujan tyypiksi on mainittu. 

Esimerkiksi, jos saamme aliohjelmassa muuttujan jonka tyyppi on `Object` ja sen arvona `"✋"`, voimme nähdä että muuttujan tyyppi on myös merkkijono.

```java
void main() {
    IO.println(onMerkkijono("✋"));
    IO.println(onMerkkijono(42));
}

boolean onMerkkijono(Object object) {
  return object instanceof String;
}
```

Vastaavasti, voimme tarkistaa itsemääritettyjen tyyppien perintäsuhteita.

(tämän esimerkin voi olla hyvä purkaa useampaan)

```java
// FILE: main.java
public class Main {
    public static void main() {
        Object opiskelija = new Opiskelija("Olli Opiskelija");
        Object opettaja = new Opettaja("Maija Opettaja");

        // muuttuja `opiskelija` on määritelmänsä mukaisesti tyyppiä `Opiskelija`
        IO.println(opiskelija instanceof Opiskelija); // true

        // luokka `Opiskelija` perii `Henkilo`-luokan (tai laajentaa sitä),
        // joten `opiskelija` on myös tyyppiä `Henkilo`
        IO.println(opiskelija instanceof Henkilo); // true

        // `opiskelija` on myös tyyppiä `Object`, sillä kaikki luokat perivät automaagisesti `Object`-luokan
        IO.println(opiskelija instanceof Object);

        // sen sijaan `opiskelija` ei ole tyyppiä `Opettaja`
        IO.println(opiskelija instanceof Opettaja); // false
        // sekä `Opettaja` että `Opiskelija` perivät `Henkilo`-luokan,
        // mutta eivät toisiaan

        // vastaavasti `opettaja` ole tyyppiä `Opiskelija`
        IO.println(opettaja instanceof Opiskelija); // false

        // perintä toimii vain yhteen suuntaan eli "alaspäin":
        Object olio = new Object();
        IO.println(olio instanceof Object); // true
        IO.println(olio instanceof Henkilo); // false
    }
}
// FILE_END
// FILE: Henkilo.java
class Henkilo {
    private String nimi;

    public Henkilo(String nimi)
    {
        this.nimi = nimi;
    }

    public String getNimi()
    {
        return nimi;
    }
}
// FILE_END
// FILE: Opiskelija.java
import java.util.ArrayList;
class Opiskelija extends Henkilo {
    ArrayList<String> kaynnissaOlevatKurssit;

    public Opiskelija(String nimi) {
        super(nimi);
        kaynnissaOlevatKurssit = new ArrayList<>();
    }

    void ilmoittauduKurssille(String kurssi) {
        kaynnissaOlevatKurssit.add(kurssi);
    }

    public void naytaKurssit(){
        String kaikkiKurssit = String.join(", ", kaynnissaOlevatKurssit);
        IO.println(this.getNimi() + " opiskelee kursseilla: " + kaikkiKurssit);
    }
}
// FILE_END
// FILE: Opettaja.java
import java.util.ArrayList;
class Opettaja extends Henkilo {
    private ArrayList<String> opetettavatKurssit;

    public Opettaja(String nimi)
    {
        super(nimi);
        this.opetettavatKurssit = new ArrayList<>();
    }

    void lisaaKurssi(String kurssi) {
        opetettavatKurssit.add(kurssi);
    }

    void naytaOpetettavatKurssit() {
        String kurssit = String.join(", ", opetettavatKurssit);
        IO.println(this.getNimi() + " opettaa kursseja: " + kurssit);
    }
}
// FILE_END
```

Seuraavaksi lyhyt varoituksen sana `instanceof`-operaattorin käytöstä olio-ohjelmoinnissa: ...

On kuitenkin niin, että `instanceof`-operaattorin käyttö tarkoittaa varsin usein sitä, ettei perintää ja polymorfismia ole hyödynnetty optimaalisella tavalla, jonka seurauksena koodiin tulee runsaasti ehtolauseita, jotka tarkistavat olion tyypin ja suorittavat sen perusteella erilaisia toimintoja. Tällöin menetetään olio-ohjelmoinnin keskeinen etu, eli se, että olioiden erilaiset toteutukset voidaan piilottaa niiden käyttäjiltä. Käytännössä ainoa, missä kyseistä operaattoria tarvitsee, on, jos käsitellään `Object`-olioita jonkin hyvin matalan tason yleisluokan kautta. 

Ehkä voisi mainita tai täsmentää, että on ohjelmointiongelmia, jotka ovat helpommin ratkaistavissa muuten kuin tiukkoja olio-ohjelmoinnin periaatteita kuten kapselointia hyödyntäen (etenkin kun ei tarvitse välittää ylläpidettävyydestä, kertakäyttökoodi tms.) — ts. olio-ohjelmointi ei ole aina paras työkalu mutta tämän tason pohdinta ei ole tämän kurssin tavoitteissa vaan nyt keskitytään hyvään ja ylläpidettävään olio-ohjelmointiin. Tässä jopa voisi olla esimerkki jostain tilanteesta, jossa `instanceof` on perusteltua käyttää verkon, esim. puun tms yksinkertaisen tietorakeen läpikäyntiin, selkeästi freimattuna kurssin ulkopuoliseksi ekstra-asiaksi.

> Kertaustehtävä kapseloinnista teoriamonivalintana
>
> Perintä ja ajonaikainen ilmentymäntarkistus mahdollistavat seuraavanlaisen täysin toimivanaohjelman rakenteen:  (esimerkki `instanceof` if:ssä) 
> - Miksi tällaista ohjelmointityyliä voisi kannattaa välttää? 
> - Miten/miksi (esimerkki `instanceof` if:ssä) rikkoo kapseloinnin? 

</details>
