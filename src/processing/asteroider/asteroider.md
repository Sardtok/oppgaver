---
title: Asteroider
level: 2
author: Sigmund Hansen
---

# Introduksjon {.intro}

Dette er en lengre oppgave hvor vi skal utvikle spillet Asteroider, basert på Atari-klassikeren Asteroids. I dette spillet flyr man et romskip rundt på skjermen og skyter ned asteroider. Når man skyter en asteroide, brytes den opp i flere mindre asteroider. De minste asteroidene går helt i stykker når man skyter dem. Om man blir truffet av en asteroide, dør man.

Først skal vi lage et romskip som man kan fly rundt på skjermen, og når man flyr ut av skjermen på en side, dukker man opp på motsatt side av skjermen. Her vil vi ha fokus på opptegning av romskipet, håndtering av tastetrykk og å flytte skipet rundt på skjermen.

Så skal vi legge til asteroider som svever rundt i rommet. Disse skal ødelegge romskipet om de kolliderer med det, og gå i stykker om man skyter dem. Når de går i stykker, deles de opp i mindre deler som kastes ut i rommet. Disse må man også passe seg for og skyte ned.

Til slutt skal vi legge en avsluttende hånd på verket. Poeng og liv skal vises, man skal få ekstraliv når man når bestemte poengsummer, og man skal ha en startskjerm som man sendes tilbake til når man taper, sånn at man kan begynne på nytt.

Det vil fortsatt være en del rom for forbedringer som man kan legge til. Lagring av rekorder, UFOer som skyter på spilleren som i klassikeren og å bruke bildefiler til grafikken isteden.

# Skipet - Opptegning {.activity}

La oss først se på hvilke egenskaper skipet skal ha. Det må i hvert fall ha en posisjon, så vi trenger et par variabler for dette *x* og *y*. Det skal også kunne snu seg rundt og fly i enhver retning på skjermen, så vi må vite hastigheten og hvilken vei skipet peker. Jeg velger her å kalle retningen for vinkel, men du kan gjerne kalle det retning. Bare vær bevisst på at det er hvilken retning skipet peker, og ikke hvilken vei det beveger seg.

```processing
float x;
float y;
float xFart;
float yFart;
float vinkel;
```

Nå som vi har variablene som er nødvendig for å tegne og flytte skipet, kan vi begynne med denne koden. La oss først sette noen startverdier for variablene, så vi kan teste koden som tegner opp og flytter skipet. Legg merke til at vinkelen settes til -90 grader, men at vi bruker `PI / 2` som 90 grader. Det er fordi vi bruker radianer og ikke grader som måleenhet for vinkler i Processing. De fleste matematiske funksjoner bruker radianer, så det gjør det enklere å bruke disse. Om man ikke kjenner radianer, finnes det en enkel måte å konvertere fra grader til radianer i Processing: `radians(-90)`, gir det samme som `-PI / 2`

```processing
void setup() {
  size(400, 400);
  x = width / 2;
  y = height / 2;
  xFart = 0;
  yFart = 0;
  vinkel = -PI / 2;
}
```

Skipet skal være en trekant som tegnes opp på den angitte plassen og er rotert i riktig vinkel. En så enkel form kan vi godt rotere selv og tegne opp ved å beregne plasseringen til hvert hjørne. Her skal vi bruke en litt enklere fremgangsmåte. Det finnes noen funksjoner i Processing som lar oss flytte, rotere og skalere opptegningen vår. Det vil si at vi kan flytte oss til dit vi skal tegne skipet, endre vinkelen på alt vi tegner og så tegne skipet uten noen regnestykker. Dette er også den enkleste måten å tegne opp bilder som skal roteres. For å flytte plasseringen, bruker vi `translate`. Så bruker vi `rotate` for å rotere. Hvis vi også trenger skalering, kan vi bruke `scale`; hvis vi vil at spillet skal kjøre i fullskjerm, er scale veldig nyttig.

```processing
void draw() {
  background(0);
  
  translate(x, y);
  rotate(vinkel);
  triangle(-10, -10, -10, 10, 20, 0);
}
```

Nå har vi tegnet opp en trekant som peker rett opp. Før vi går videre skal vi se litt på koden, så vi er sikre på at vi forstår hva den gjør. Først kan du prøve forskjellige verdier for `x`, `y` og `vinkel`, så kan du se resultatene. `PI` er 180 grader, `2 * PI` er 360 grader, `0.5 * PI` eller `PI / 2` er 90 grader, osv.

La oss ta en rask titt på de tre siste setningene i `draw`. Den første, `translate(x, y)`, flytter utgangspunktet for opptegningen fra øverst til venstre til plasseringen angitt av `x` og `y`. I dette tilfellet betyr det til midten av vinduet. Nå vil alle koordinater ta utgangspunkt i denne plasseringen. Det betyr at `-10` i X (horisontalt) vil bli `10` piksler til venstre for midten, og `-10` i Y blir `10` piksler over midten. Hvis du setter vinkelen til `0`, kan du se at ett hjørne er `10`piksler til venstre for og `10` piksler over midten. Det neste er `10` piksler til venstre for og `10` piksler under midten. Det siste hjørnet av trekanten er `20` piksler til høyre for midten. Før vi tegner opp trekanten, roterer vi også all opptegningen med `rotate(vinkel)`. Det som da skjer, er at alle koordinater vi bruker etter dette vil roteres. I dette tilfellet hvor vi roterer 90 grader til venstre, blir det som var høyre, opp isteden. Hvis man vil regne det ut for hånd, kan man multiplisere X-koordinatet med cosinus av vinkelen og Y-koordinatet med sinus av vinkelen.

Rekkefølgen man utfører operasjoner som `translate`, `rotate` og `scale` påvirker hva som skjer. Hver gang man kaller en av disse funksjonene, endres utgangspunktet og det neste funksjonskallet vil bruke det nye utgangspunktet. Det finnes funksjoner for å lagre dette utgangspunktet, men det finnes noen begrensninger. Man kan bare lagre 32 slike tilstander, og man må passe på at man ikke lar lagrede tilstander bli liggende. Dette innebærer vanligvis at man lagrer rett før man tegner opp noe, endrer det, tegner opp og går tilbake til tilstanden man lagret før opptegningen. Vi skal se på dette når vi legger inn asteroidene.

# Skipet - Styring {.activity}

Før du fortsetter med styringen, kan det være lurt å justere tilbake variablene du endret for å teste opptegningen.

For å best mulig kunne utnytte tastaturet, skal vi legge til noen variabler for å huske hvilke knapper som er trykket inn. Vi trenger en for å snu til venstre, en til høyre, en for å fly framover og en for å skyte:

```processing
boolean venstre;
boolean hoyre;
boolean fram;
boolean skyt;
```

I tillegg må vi reagere på at brukeren trykker inn en av de tilhørende tastene. Tastetrykk kan vi håndtere i `keyPressed`. Her tillater vi spilleren å bruke både WASD og piltastene for å styre og mellomrom, CTRL, linjeskift og J kan alle brukes til å skyte.

```processing
void keyPressed() {
  if (key == 'w' || keyCode == UP) {
    fram = true;
  } else if (key == 'a' || keyCode == LEFT) {
    venstre = true;
  } else if (key == 'd' || keyCode == RIGHT) {
    hoyre = true;
  } else if (key == ' ' || key == 'j' || keyCode == ENTER || keyCode == CONTROL) {
    skyt = true;
  }
}
```

Spillet må også merke om spilleren slipper en tast, sånn at ikke romskipet snurrer rundt eller skyter for alltid, men bare når spilleren ønsker det. Den enkleste måten å gjøre det på er å kopiere `keyPressed`-funksjonen vi nettopp lagde, men bytte ut `true` med `false` og endre navnet til `keyReleased`:

```processing
void keyReleased() {
  if (key == 'w' || keyCode == UP) {
    fram = false;
  } else if (key == 'a' || keyCode == LEFT) {
    venstre = false;
  } else if (key == 'd' || keyCode == RIGHT) {
    hoyre = false;
  } else if (key == ' ' || key == 'j' || keyCode == ENTER || keyCode == CONTROL) {
    skyt = false;
  }
}
```

## Snu skipet {.check}

Nå har vi kontroll på hvilke taster som er i trykket inn, men vi mangler fortsatt å oppdatere romskipet slik at det flyr rundt på skjermen. La oss begynne med å få romskipet til å snurre rundt. Vi legger til en variabel for hvor fort romskipet skal snu når man styrer til venstre eller høyre. Fordi denne verdien ikke skal endre seg mens spillet kjører, setter vi den til å være `final` og gir den et navn med store bokstaver. Slike variabler kalles gjerne konstanter siden de ikke kan endres mens programmet kjører.

```processing
final float SNURREFART = PI / 60;
```

For å unngå å putte all koden i `draw` lager vi en ny funksjon for å oppdatere vinkelen og posisjonen til romskipet, `oppdaterSkip`. På denne måten kan vi skjønne hva koden gjør uten å lese den, og i tillegg blir det lettere å lese fordi hver funksjon er kort. La oss legge den til og oppdatere vinkelen basert på tastene som er trykket inn. Vi må også kalle den nye funksjonen i `draw` sånn at skipet oppdateres for hvert bilde som tegnes.

```processing
void oppdaterSkip() {
  if (venstre) {
    vinkel -= SNURREFART;
  }
  if (hoyre) {
    vinkel += SNURREFART;
  }
}

void draw() {
  oppdaterSkip();
  
  background(0);
  ...
}
```

## Fly {.check}

Nå har vi et romskip som kan snu seg rundt i en hastighet på ca. 180 grader per sekund. Da kan vi fortsette med å fly framover. Dette innebærer en liten liste med ting som må gjøres:

+ Beregne aksellerasjonen ut fra vinkelen og endre hastigheten ut fra den.
+ Flytte skipet til motsatt side av vinduet når det flyr ut av vinduet.
+ Begrense hastigheten på skipet for å gjøre det enklere å spille.
+ Tegne noen flammer fra motoren på skipet når det gir gass.

La oss begynne med å beregne aksellerasjonen til sidene og opp og ned ut fra vinkelen. Vi legger til enda en konstant for hvor fort skipet aksellerer:

```processing
final float AKSELLERASJON = 0.1;
```

Så bruker vi cosinus av vinkelen for å regne ut aksellerasjonen langs X-aksen (til sidene) og sinus for Y-aksen (opp og ned). Ved å gange resultatet med aksellerasjonskonstanten, vil aksellerasjonen bli lik konstanten, men peke i riktig retning. Så legger vi aksellerasjonen til farten og til slutt må vi legge farten til posisjonen:

```processing
void oppdaterSkip() {
  ...
  if (fram) {
    xFart += cos(vinkel) * AKSELLERASJON;
    yFart += sin(vinkel) * AKSELLERASJON;
  }
  
  x += xFart;
  y += yFart;
}
```

Nå har du et romskip som veldig fort forsvinner ut av vinduet og forblir borte. Så la oss fikse det ved å flytte skipet til motsatt side av vinduet om det flyr ut på en av sidene. For å gjøre dette enklest mulig, flytter vi skipet når det har beveget seg så langt ut at det ikke kan være innenfor vinduet lenger. Det må da også dukke opp like langt utenfor vinduet på motsatt side. Siden skipets lengste spiss stikker `20` piksler ut, flytter vi skipet når det har kommet `20` piksler utenfor vinduet, og vi flytter det slik at det begynner `20` piksler utenfor vinduet i motsatt ende. Da må vi huske at `-20 + width + 20 = width`, så vi må legge til `width + 40` og ikke `width + 20`. Vi legger til en ny konstant for hvor langt ut denne kanten av rommet befinner seg, eller hvor stor margen til vinduet er:

```processing
final float MARG = 20;
```

Og så legger vi til koden som flytter skipet:

```processing
void oppdaterSkip() {
  ...
  if (x < -MARG) {
    x += width + MARG * 2;
  } else if (x > width + MARG) {
    x -= width + MARG * 2;
  }
  
  if (y < -MARG) {
    y += height + MARG * 2;
  } else if (y > height + MARG) {
    y -= height + MARG * 2;
  }
}
```

Nå går det an å fly rundt i rommet uten at man forsvinner for evig og alltid. Det er ganske vanskelig å styre siden man kan ende opp med så ekstrem fart at man mister all kontroll. Så vi må legge til en begrensning på hvor fort man kan fly. Først velger vi en maksimal hastighet, og så bruker vi Pytagoras til å finne den faktiske hastigheten. Hvis hastigheten er høyere enn farten vi har valgt, må vi justere farten uten å endre retningen. Først legger vi til den nye konstanten:

```processing
final float MAKS_FART = 3.0;
```

For å justere ned hastigheten, regner vi ut hva den maksimale farten er i forhold til hva farten har blitt. Så ganger vi farten i hver retning med dette forholdstallet. Vi legger til litt kode i if-setningen som endrer hastigheten når man styrer framover:

```processing
void oppdaterSkip() {
  ...
  if (fram) {
    xFart += cos(vinkel) * AKSELLERASJON;
    yFart += sin(vinkel) * AKSELLERASJON;
    
    float fart = sqrt(xFart * xFart + yFart * yFart);
    if (fart > MAKS_FART) {
      float forhold = MAKS_FART / fart;
      xFart *= forhold;
      yFart *= forhold;
    }
  }
  ...
}
```

Da gjenstår det bare å legge til noen flammer eller liknende bak skipet når man gir gass. Det finnes utrolig mange måter å tegne slike flammer. Derfor bør du selv velge hvordan du vil at disse flammene skal se ut. Likevel får du se en enkel løsning her:

```processing
void draw() {
  oppdaterSkip();
  
  background(0);
  
  translate(x, y);
  rotate(vinkel);

  noStroke();
  fill(255);
  triangle(-10, -10, -10, 10, 20, 0);
  
  if (fram) {
    stroke(255);
    noFill();
    ellipse(-15, 0, 10, 10);
    line(-20, 0, -25, 0);
  }
}
```

Merk at siden vi slår av fylling av former som tegnes opp, og endrer linjefargen til hvit når vi tegner opp flammen, så må vi sette fyllfargen tilbake til hvit og slå av opptegning av omrisset.

# Asteroider {.activity}

Nå som vi har et romskip vi kan fly rundt, er det på tide å få lagt inn noen asteroider som vi kan skyte på. Vi vil ha flere enn en, så vi må bruke lister av noe slag til å holde rede på disse asteroidene. For å holde koden ryddig skal vi også bruke en klasse for asteroidene som vi legger i en egen fane. Klasser lar oss samle data og funksjonalitet som hører sammen i en egen atskilt del. De lar oss også lage flere eksemplarer av ting som har samme type data og oppfører seg på samme måte; disse eksemplarene kalles objekter.

+ Lag en ny fane ved å trykke på den lille nedoverpilen og velg "New Tab" (Ctrl + Shift + N eller Cmd + Shift + N).
+ Gi den navnet "Asteroide".

Faner i Processing er et merkelig lite triks. Innholdet i alle fanene slåes sammen til en stor fil før programmet ditt lages. Det er derfor viktig at innholdet i hver fane er atskilte deler: en eller flere hele funksjoner, hele klasser eller variabeldeklarasjoner (variabeldeklarasjoner er de setningene som lager en ny variabel).

Nå som vi har en fane til asteroide-koden, kan vi begynne å lage asteroide-klassen vår her. Alle asteroider har en posisjon, en hastighet og en størrelse. Så vi begynner med å lage en klasse med disse variablene, og med en konstruktør som setter disse:

```processing
class Asteroide {
  float x;
  float y;
  float xFart;
  float yFart;
  float radius;
  
  Asteroide(float x, float y, float xFart, float yFart, float radius) {
    this.x = x;
    this.y = y;
    this.xFart = xFart;
    this.yFart = yFart;
    this.radius = radius;
  }
}
```

Dette er kanskje den første oppgaven hvor du bruker klasser, og da er det sikkert mye nytt i koden ovenfor, så la oss ta en titt på det som står her. Når vi skal lage en ny klasse, begynner vi med `class Klassenavn` etterfulgt av innholdet i klassen i krøllparenteser.

Inne i klassen ser vi to ting, først fem variabeldeklarasjoner som sier hvilke variabler som skal finnes i Asteroide-objekter. Disse er etterfult av en konstruktør. Konstruktører er en spesielle funksjon som brukes når man lager objekter av en klasse. Ofte ønsker vi å sette variabler i et objekt når det lages, og det gjøres da inne i konstruktøren.

Konstruktøren er en spesiell funksjon med samme navn som klassen og uten returtype foran navnet. Her har vi også valgt å ha med parametere for alle variablene i klassen slik at vi kan sette dem. Om du ikke har sett funksjoner med parametere bli deklarert før, så er det bare variabler som deklareres i parentesene til funksjonen med komma mellom; verdiene deres settes når funksjonen kalles som du har sett i eksempler som kallet på `triangle` når vi tegner skipet. `this` brukes for å skille mellom variablene i funksjonen og variablene som finnes i objektet som opprettes. Det er et spesielt ord som gir oss tilgang til objektet funksjonen kjøres for. `this.x = x;` sier at objektets `x` skal settes til verdien til `x` i funksjonskallet.

Nå har vi sett på koden, men vi har ikke sett hva den gjør eller hvordan vi kan bruke den ennå. La oss legge til en asteroide og tegne den opp.

+ Åpne den andre fanen der koden for skipet finnes.
+ Lag en variabel med type Asteroide:
  
  ```processing
  Asteroide ast;
  ```
  
+ Lag en asteroide og sett verdien til variabelen vi akkurat lagde:
  
  ```processing
  void setup() {
    size(400, 400);
    x = width / 2;
    y = height / 2;
    xFart = 0;
    yFart = 0;
    vinkel = -PI / 2;
  
    ast = new Asteroide(50, 50, 3, 2, 20);
  }
  ```
  
+ Nullstill forflytning og rotasjon med `resetMatrix` og tegn opp asteroiden:
  
  ```processing
  void draw() {
    oppdaterSkip();
    
    background(0);
    
    translate(x, y);
    rotate(vinkel);
  
    noStroke();
    fill(255);
    triangle(-10, -10, -10, 10, 20, 0);
    
    if (fram) {
      stroke(255);
      noFill();
      ellipse(-15, 0, 10, 10);
      line(-20, 0, -25, 0);
    }
    
    resetMatrix();
    
    // Tegn asteroiden
    ellipse(ast.x, ast.y, ast.radius * 2, ast.radius * 2);
  }
  ```

Nå kan vi se at det tegnes en sirkel `50` piksler inn i vinduet fra venstre og toppen. Den har en radius på `20` piksler eller en diameter på `40` piksler. Men asteroiden beveger seg ikke, selv om vi ga den en fart på `3` piksler til høyre per bilde og `2` piksler ned per bilde. Det er fordi vi bruker ikke farten til å endre posisjonen.

Skal vi legge til kode for å flytte asteroiden inn i `draw`? Hva med når asteroiden forsvinner ut av vinduet? Her risikerer vi at det kan bli mye kode som må inn her også, men vi har jo allerede en ny fane for å putte koden til asteroiden i. Så la oss flytte opptegningskoden til asteroiden inn der sammen med oppdatering av posisjonen.

+ Bytt til Ateroide-fanen.
+ Lag en `draw`-funksjon i `Asteroide`-klassen:
  
  ```processing
  class Asteroide {
    ...
    void draw() {
      x += xFart;
      y += yFart;
      
      ellipse(x, y, radius * 2, radius * 2);
    }
  }
  ```
  
  Legg merke til at vi slipper å skrive `ast.` foran alle variabelnavnene. Ellers er den eneste endringen fra det vi hadde i programmets `draw` at vi oppdaterer posisjonen til asteroiden.
  
+ Bytt til den andre fanen.
+ Bytt ut opptegningen av asteroiden med et kall på asteroidens `draw`:
  
  ```processing
  void draw() {
    oppdaterSkip();
    
    background(0);
    
    translate(x, y);
    rotate(vinkel);
  
    noStroke();
    fill(255);
    triangle(-10, -10, -10, 10, 20, 0);
    
    if (fram) {
      stroke(255);
      noFill();
      ellipse(-15, 0, 10, 10);
      line(-20, 0, -25, 0);
    }
    
    resetMatrix();
    
    // Tegn asteroiden
    ast.draw();
  }
  ```

Nå svever asteroiden gjennom rommet og forsvinner ut på høyre side av vinduet slik vi skulle forvente.

## Speiling om vinduskantene

Men den ser jo ikke akkurat ut som en asteroide, og den forsvinner også ut av vinduet og blir borte. Så la oss ordne dette.

Den enkleste biten blir å rette opp at den ikke dukker opp på motsatt side av vinduet. Det har vi allerede kode for, så nå må vi bare finne en måte å gjenbruke den samme koden for skipet vårt og asteroidene.

+ Flytt koden for å finne nye koordinater ut av `oppdaterSkip`:
  
  ```processing
  float beregnKoordinat(float koordinat, float fart, float lengde, float marg) {
    koordinat += fart;
    
    if (koordinat < -marg) {
      koordinat += lengde + marg * 2;
    } else if (koordinat > lengde + marg) {
      koordinat -= lengde + marg * 2;
    }
    
    return koordinat;
  }
  ```
  
  Dette er kanskje første gangen du lager en funksjon som har en verdi som resultat. Legg merke til at istedenfor `void`, står det `float` foran navnet på funksjonen. Det betyr at denne funksjonen gir et flyttall som resultat. Det gjør at vi må ha med en retur-seting (`return koordinat;` i dette tilfellet). Det er retur-setninger som sier hva resultatet av en funksjon er. Dette er kanskje også første gang du lager en funksjon som tar imot parametere med unntak av konstruktøren du skrev tidligere. Denne tar fire stykker og alle har typen `float`. Disse kan du bruke som vanlige variabler inne i funksjonen, men de lever bare inn i funksjonen.

+ Ta i bruk den nye funksjonen i `oppdaterSkip`:
  
  ```processing
  void oppdaterSkip() {
    if (venstre) {
      vinkel = beregnKoordinat(vinkel, -SNURREFART, 2 * PI, 0);
    }
    if (hoyre) {
      vinkel = beregnKoordinat(vinkel, SNURREFART, 2 * PI, 0);
    }
    
    if (fram) {
      xFart += cos(vinkel) * AKSELLERASJON;
      yFart += sin(vinkel) * AKSELLERASJON;
      
      float fart = sqrt(xFart * xFart + yFart * yFart);
      if (fart > MAKS_FART) {
        float forhold = MAKS_FART / fart;
        xFart *= forhold;
        yFart *= forhold;
      }
    }
    
    x = beregnKoordinat(x, xFart, width, MARG);
    y = beregnKoordinat(y, yFart, height, MARG);
  }
  ```
  
  Oi, se der. Vi kunne til og med bruke den nye funksjonen for å beregne vinkelen til skipet. Det virker kanskje unødvendig, men det beskytter oss mot noen merkelige feil som kan opptre om man snurrer i en retning lenge nok. Disse feilene ville nok ikke skjedd før etter mange mange timer, men når vi først har lagd denne funksjonen kan vi like godt utnytte den.
  
+ Ta i bruk den nye funksjonen for å flytte asteroiden:
  
  ```processing
  class Asteroide {
    ...
    void draw() {
      x = beregnKoordinat(x, xFart, width, MARG);
      y = beregnKoordinat(y, yFart, height, MARG);
      
      ellipse(x, y, radius * 2, radius * 2);
    }
  }
  ```
  
  Sånn, da oppfører skipet og asteroiden seg likt når det kommer til kantene av vinduet.

## Utseendet til asteroiden

La oss nå rette opp i utseendet til asteroiden; den er altfor jevn. For å rette opp i dette skal vi heller tegne den opp som en mangekant. Vi begynner med en regulær mangekant, og så legger vi til ekstra ujevnheter etterpå. Stegene vi så skal gjøre er da:

+ Sett opp fyll og omriss så det passer for opptegning av asteroiden.
+ Flytt koordinatsystemet.
+ Tegn en regulær mangekant med en løkke. Vi begynner med en tikant, men antallet kanter bør bestemmes av størrelsen til asteroiden når den opprettes.
+ Nullstill koordinatsystemet med `resetMatrix`.

```processing
class Asteroide {
  ...
  void draw() {
    x = beregnKoordinat(x, xFart, width, MARG);
    y = beregnKoordinat(y, yFart, height, MARG);
    
    noFill();
    stroke(255);
    
    translate(x, y);
    
    beginShape();
    for (int i = 0; i < 10; i++) {
      vertex(radius * cos(i * PI / 5), radius * sin(i * PI / 5));
    }
    endShape(CLOSE);
    
    resetMatrix();
  }
}
```

Nå har du en kantete, men fortsatt nesten sirkulær asteroide. Vi trenger litt variasjon for at den skal se mer naturlig ut. Da vil vi variere vinkelen mellom hvert hjørne og avstanden fra midten av asteroiden til hjørnet. Dette må vi lagre et sted, sånn at det ikke forandrer seg hele tiden.

+ Lag variabler for en liste for X-koordinatene til punktene, og en for Y-koordinatene.
  
  ```processing
  class Asteroide {
    float[] punkterX;
    float[] punkterY;
    ...
  }
  ```

+ Lag et antall punkter basert på størrelsen til asteroiden. Flytt punktene litt vekk fra plassene langs omrisset til sirkelen.
  
  ```processing
  class Asteroide {
    ...
    Asteroide(float x, float y, float xFart, float yFart, float radius) {
      ...
      int punkter = 10;
      if (radius <= 10) {
        punkter = 5;
      } else if (radius <= 15) {
        punkter = 7;
      }
      
      punkterX = new float[punkter];
      punkterY = new float[punkter];
      for (int i = 0; i < punkter; i++) {
        float avstand = random(0.75 * radius, 1.25 * radius);
        float vinkel = i * 2 * PI / punkter + random(-0.5 * PI / punkter, 0.5 * PI / punkter);
        punkterX[i] = avstand * cos(vinkel);
        punkterY[i] = avstand * sin(vinkel);
      }
    }
  }
  ```

+ Endre opptegningen til å bruke disse punktene istedenfor å tegne en helt regulær mangekant.
  
  ```processing
  class Asteroide {
    ...
    void draw() {
      x = beregnKoordinat(x, xFart, width, MARG);
      y = beregnKoordinat(y, yFart, height, MARG);
      
      noFill();
      stroke(255);
      
      translate(x, y);
      
      beginShape();
      for (int i = 0; i < punkterX.length; i++) {
        vertex(punkterX[i], punkterY[i]); // Bare denne linjen er endret
      }
      endShape(CLOSE);
      
      resetMatrix();
    }
  }
  ```

+ Prøv ut forskjellige størrelser på asteroiden og se om du vil gjøre noen endringer i hvordan asteroidene skal se ut.
  
  + Bør antall punkter være annerledes?
  + Er det nok variasjon i avstanden fra sentrum, eller er det for mye?
  + Hva med vinklene mellom hvert hjørne?

+ Det ser litt rart ut når asteroiden ikke snurrer.
  
  + Legg til en variabel for vinkelen til asteroiden.
  + Legg til en variabel for hvor fort asteroiden skal snurre.
  + Sett en tilfeldige snurrefart for asteroiden i konstruktøren, for eksempel mellom `-SNURREFART` og `SNURREFART`.
  + Oppdater vinkelen i `draw` og roter asteroiden på samme måte som ble gjort for skipet.

## Flere asteroider

En enslig asteroide er jo ikke særlig spennende. Det blir jo altfor lett om spilleren bare skal skyte den ene. Så vi trenger å lage en liste med asteroider som skal brukes, og så tegne opp alle sammen.

+ Bytt ut den enslige asteroiden med en liste (fjern linjen: `Asteroide ast;`):
  
  ```processing
  Asteroide[] asteroider;
  ```
  
+ Lag noen flere asteroider:
  
  ```processing
  void setup() {
    size(400, 400);
    x = width / 2;
    y = height / 2;
    xFart = 0;
    yFart = 0;
    vinkel = -PI / 2;
    
    asteroider = new Asteroide[3];
    for (int i = 0; i < asteroider.length; i++) {
      float retning = random(0, 2 * PI);
      float fart = random(MAKS_FART);
      asteroider[i] = new Asteroide(random(width), random(height), fart * cos(retning), fart * sin(retning), 20);
    }
  }
  ```
  
+ Tegn opp asteroidene:
  
  ```processing
  void draw() {
    oppdaterSkip();
    
    background(0);
    
    translate(x, y);
    rotate(vinkel);
  
    noStroke();
    fill(255);
    triangle(-10, -10, -10, 10, 20, 0);
    
    if (fram) {
      stroke(255);
      noFill();
      ellipse(-15, 0, 10, 10);
      line(-20, 0, -25, 0);
    }
    
    resetMatrix();
    
    // Tegn asteroidene
    for (Asteroide ast : asteroider) {
      ast.draw();
    }
  }
  ```

### For-each {.protip}

Her ser vi en annen variant av en for-løkke, som kan brukes i visse tilfeller. Man kan ikke bruke denne varianten hvis man skal bruke flere lister eller endre på innholdet i listen. Hvis man bare skal lese innholdt i listene, fungerer denne varianten og er litt enklere å skrive. Løkken som er brukt over, tilsvarer den følgende løkken:

```processing
for (int i = 0; i < asteroider.length; i++) {
  Asteroide ast = asteroider[i];
  ast.draw();
}
```

Denne typen løkker kan brukes med andre typer datastrukturer også, så det gjør ting enklere ved at man bruker det likt uansett. Ordet datastrukterer brukes om alle ting som lar oss organisere mengder med data. Dette kan være lister, men også andre strukturer kalt mengder, trær, grafer, køer og liknende. Skal man jobbe med disse kan man nesten alltid bruke en løkke som den over, for å gjøre noe arbeid med alle dataene organisert i strukturen.

# PANG PANG! {.activity}

Men dette er ikke et spill helt ennå. Det finnes ikke noe mål. Vi skulle jo kunne skyte asteroidene, og om de krasjet i skipet, skulle jo det gå i stykker.

## Kuleregn {.check}

Akkurat som asteroidene, vil vi nå lage en ny klasse for kuler. Den skal fly et stykke framover før den forsvinner, og spilleren kan kun skyte et lite antall kuler, la oss si tre. Hvis det er skutt tre kuler, må spilleren vente til den første kulen er borte før de kan skyte på nytt.

+ Lag en ny fane for Kule-klassen.
+ Kuler har en posisjon og en retning i tillegg til en begrenset levetid. Vi tar også med hastighet, selv om denne kunne vært regnet ut ved hjelp av vinkelen.
  
  ```processing
  class Kule {
    float x;
    float y;
    float xFart;
    float yFart;
    float vinkel;
    int levetid;
  }
  ```

+ Spilleren må kunne avfyre kuler. Dette kunne vært gjort ved å lage nye kuler hver gang. Isteden lager vi alle kulene når spillet starter, og så setter vi variablene når de avfyres. Levetiden brukes til å holde rede på om de skal tegnes og om de kan skade asteroider.
  
  ```processing
  class Kule {
    ...
    
    void avfyr(float x, float y, float vinkel) {
      this.x = x;
      this.y = y;
      this.vinkel = vinkel;
      
      xFart = 5 * cos(vinkel);
      yFart = 5 * sin(vinkel);
      levetid = 60;
    }
  }
  ```

+ Kulen tegnes som en enkel strek, som stikker litt ut bak og litt ut foran midten. Vi må også telle ned levetiden og ikke gjøre noe når kulens levetid har gått ut. Sist, men ikke minst, må vi oppdatere posisjonen til kulen.
  
  ```processing
  class Kule {
    ...
    
    void draw() {
      if (levetid <= 0) {
        return;
      }
      
      levetid--;
      
      x = beregnKoordinat(x, xFart, width, MARG);
      y = beregnKoordinat(y, yFart, height, MARG);
      
      stroke(255);
      
      translate(x, y);
      rotate(vinkel);
      
      line(-3, 0, 3, 0);
      
      resetMatrix();
    }
  }
  ```

+ Vi må legge til noen kuler i spillet og en variabel for å telle ned hvor lenge det er til neste gang vi kan skyte, sånn at det blir et lite mellomrom mellom hver kule.
  
  ```processing
  Kule[] kuler = new Kule[3];
  int skyteNedtelling;
  
  void setup() {
    size(400, 400);
    x = width / 2;
    y = height / 2;
    xFart = 0;
    yFart = 0;
    vinkel = -PI / 2;
    
    asteroider = new Asteroide[3];
    for (int i = 0; i < asteroider.length; i++) {
      float retning = random(0, 2 * PI);
      float fart = random(MAKS_FART);
      asteroider[i] = new Asteroide(random(width), random(height), fart * cos(retning), fart * sin(retning), 20);
    }
    
    for (int i = 0; i < kuler.length; i++) {
      kuler[i] = new Kule();
    }
  }
  ```

+ Når spilleren trykker på skyteknappen, må det avfyres en kule hvis det er lenge nok siden forrige skudd, og det ikke er tre kuler på skjermen fra før.
  
  ```processing
  void oppdaterSkip() {
    if (venstre) {
      vinkel = beregnKoordinat(vinkel, -SNURREFART, 2 * PI, 0);
    }
    if (hoyre) {
      vinkel = beregnKoordinat(vinkel, SNURREFART, 2 * PI, 0);
    }
    
    if (fram) {
      xFart += cos(vinkel) * AKSELLERASJON;
      yFart += sin(vinkel) * AKSELLERASJON;
  
      float fart = sqrt(xFart * xFart + yFart * yFart);
      if (fart > MAKS_FART) {
        float forhold = MAKS_FART / fart;
        xFart *= forhold;
        yFart *= forhold;
      }
    }
    
    skyteNedtelling--;
    if (skyt && skyteNedtelling <= 0) {
      for (Kule kule : kuler) {
        if (kule.levetid <= 0) {
          kule.avfyr(x, y, vinkel);
          skyteNedtelling = 15;
          break;
        }
      }
    }
    
    x = beregnKoordinat(x, xFart, width, MARG);
    y = beregnKoordinat(y, yFart, height, MARG);
  }
  ```
  
  Her er det lagt inn en ny løkke mot slutten av `oppdaterSkip`. I denne løkken har vi brukt `break` til å avslutte løkken når vi har funnet en kule å avfyre. `break` stopper løkken fra å kjøre videre.

+ Til slutt i `draw`, må vi også tegne opp kulene.
  
  ```processing
  void draw() {
    oppdaterSkip();
    
    background(0);
    
    translate(x, y);
    rotate(vinkel);
  
    noStroke();
    fill(255);
    triangle(-10, -10, -10, 10, 20, 0);
    
    if (fram) {
      stroke(255);
      noFill();
      ellipse(-15, 0, 10, 10);
      line(-20, 0, -25, 0);
    }
    
    resetMatrix();
    
    // Tegn kulene
    for (Kule kule : kuler) {
      kule.draw();
    }
    
    // Tegn asteroidene
    for (Asteroide ast : asteroider) {
      ast.draw();
    }
  }
  ```

## Krasj-bom-bang {.check}

Nå som det regner kuler over hele skjermen, er det på tide at det skjer noe når en kule treffer en asteroide. Å sjekke om to objekter i et spill kolliderer eller overlapper, kalles kollisjonssjekking. Det finnes mange måter å sjekke kollisjoner på, som tar utgangspunkt i forskjellige geometriske egenskaper. Firkanter som ikke er rotert er det enkleste å sjekke. Sirkler er også veldig lett siden vi da bare kan basere oss på avstanden mellom sirklene. Andre geometriske former blir noe mer komplisert.

Så her må vi ta et valg, skal kollisjonssjekkingen være nøyaktig eller er det nok å sjekke for sirkler? For kuler holder sirkelmetoden. Hvis vi skal bruke den samme metoden for skipet, bør vi bruke to eller tre små sirkler. Ellers kan det bli mange kollisjoner der det ikke er noen, altså at skipet ikke er borti en asteroide, men eksploderer likevel. Her kommer vi bare til å bruke sirkelkollisjoner for begge deler, men om du vil ha en mer nøyaktig kollisjonssjekk kan du lese om: **TODO: sett inn algoritme for konvekse polygoner her, kontroller at artikkelen har med triangulering eller andre konvekseringsteknikker for konkave asteroider**.

La oss først håndtere at skudd treffer asteroider. Her må vi gjøre flere ting, sjekke om det er en kollisjon. Om et skudd treffer en asteroide, må asteroiden gå i stykker. Da deles den i to om den er stor, eller forsvinner om den er liten. Skuddet må også forsvinne.

+ Sjekk etter kollisjon og fjern skuddet ved å sette levetiden til `0` hvis det skjer:
  
  ```processing
  class Asteroide {
    ...
    void draw() {
      if (levetid <= 0) {
        return;
      }
      
      levetid--;
      
      x = beregnKoordinat(x, xFart, width, MARG);
      y = beregnKoordinat(y, yFart, height, MARG);
      
      // Kollisjonssjekking etter at alle ting er flyttet
      for (Kule k : kuler) {
        if (kollidererMed(k)) {
          k.levetid = 0;
          // Her må vi legge til at asteroiden skal bli ødelagt
          // Vi har en del utfordringer å ta hensyn til før vi kan gjøre dette
          return;
        }
      }
      
      stroke(255);
      
      translate(x, y);
      rotate(vinkel);
      
      line(-3, 0, 3, 0);
      
      resetMatrix();
    }
    
    boolean kollidererMed(Kule k) {
      if (k.levetid <= 0) {
        return false;
      }
      
      float avstandX = k.x - x;
      float avstandY = k.y - y;
      float kvadratiskAvstand = avstandX * avstandX + avstandY * avstandY;
      float maksAvstand = radius + 3;
      
      return kvadratiskAvstand < (maksAvstand * maksAvstand);
    }
  }
  ```
  
  Nå lurer du kanskje på hvordan `kollidererMed` fungerer. Hvis kulens levetid har løpt ut, kan ingenting kollidere med den, så da returnerer vi `false` (usann) uten å gjøre noen beregninger. Så bruker vi Pytagoras' læresetning for å bestemme om kulen og asteroiden kolliderer. Vi bruker avstandene langs X- og Y-aksene som katetene i en rettvinklet trekant. Så beregner vi kvadratet til hypotenusen, altså kvadratet av avstanden mellom kulen og asteroiden. Så legger vi sammen radiene til kulen og asteroiden, hvis de to er nærmere hverandre enn dette, da kolliderer de. Til slutt kunne vi gjort en av to ting, vi kunne regnet ut kvadratroten av `kvadratiskAvstand` og sammenliknet det med `maksAvstand`, eller vi kunne regnet ut kvadratet av `maksAvstand` og sammenliknet det med `kvadratiskAvstand`. Å regne ut kvadratroten av noe tar lengre tid for maskinen enn å regne ut kvadratet av noe, så når vi kan velge, velger vi heller det siste. Valget har lite å si i et lite spill som dette, men om det var tusenvis av skudd og asteroider, ville det hatt stor betydning.

  Den siste setningen `return kvadratiskAvstand < (maksAvstand * maksAvstand)` virker kanskje uvant, men en liknende `return` har du sett i `beregnKoordinat`. Her sammenlikner vi to tall. Resultatet er enten sant eller usant, og denne funksjonen returnerer nettopp en boolsk verdi. Vi regner også ut kvadratet av `maksAvstand` siden det er det vi skal sammenlikne med. For at det skal bli tydelig er det satt parenteser rundt regnestykket, men det er ikke påkrevd. 

+ Når en stor eller middels stor asteroide blir ødelagt, skal den deles i to mindre asteroider. Når en asteroide blir borte, og to nye oppstår, trenger vi plass til en ekstra asteroide. Et problem er at den typen liste vi har brukt, en *array*, har en fast størrelse. Da finnes det en del forskjellige alternativer: bruk en annen type liste som kan vokse, skriv kode for å utvide listen selv eller bruke ferdige funksjoner i Processing for å utvide listen. Vi skal her gå for det første alternativet. Bytt ut listetypen:
  
  ```processing
  ArrayList<Asteroide> asteroider; // ArrayList istedenfor en array
  ...
  void setup() {
    size(400, 400);
    x = width / 2;
    y = height / 2;
    xFart = 0;
    yFart = 0;
    vinkel = -PI / 2;
    
    asteroider = new ArrayList<Asteroide>(); // Oppretting av denne typen lister ser litt annerledes ut
    for (int i = 0; i < 3; i++) {
      float retning = random(0, 2 * PI);
      float fart = random(MAKS_FART);
      
      // Innsetting er også litt annerledes
      asteroider.add(new Asteroide(random(width), random(height), fart * cos(retning), fart * sin(retning), 20));
    }
    
    for (int i = 0; i < kuler.length; i++) {
      kuler[i] = new Kule();
    }
  }
  ```
  
  Over er det fire linjer som har blitt endret. Den første når variabelen deklareres har en annen type enn tidligere `ArrayList<Asteroide>`. `ArrayList` er en liste som bruker en *array* til å putte elementene i, men håndterer endringer i antall elementer og fjerning på en effektiv måte. Den neste er opprettelsen av listen inne i `setup` rett før løkken som fyller listen. Denne setningen er nesten lik som når vi oppretter asteroider eller kuler. Så har vi endret antall ganger løkken skal kjøre, sånn at den ikke er avhengig av listen når vi lager asteroidene første gang. Til slutt er setningen som legger til asteroidene i listen endret: nå bruker vi `ArrayList.add` istedenfor.
  
  Hvis du tester å kjøre programmet, skal alt fungere akkurat som før.

+ Nå må vi endre på inneholdet i listen hvis en asteroide blir skutt i stykker. Det må gjøres i to steg. Vi kan ikke legge til elementer i listen mens vi løper gjennom den, altså inne i for-løkken i `draw`. Så vi må gjøre noen endringer. Først legger vi til følgende `import`-setning helt først i programmet:
  
  ```processing
  import java.util.Iterator;
  ```
  
  Denne gir oss tilgang til `Iterator`-klassen som er det som brukes til for-each-løkker som løper gjennom andre datastrukturer enn *arrayer*. Det neste steget er å få `Asteroide.draw` til å gi beskjed når asteroiden blir ødelagt, så vi endrer `draw` til å returnere en boolsk verdi:
  
  ```processing
  class Asteroide {
    ...
    boolean draw() {
      x = beregnKoordinat(x, xFart, width, MARG);
      y = beregnKoordinat(y, yFart, height, MARG);
      vinkel = beregnKoordinat(vinkel, vinkelFart, 2 * PI, 0);
      
      // Kollisjonssjekking etter at alle ting er flyttet
      for (Kule k : kuler) {
        if (kollidererMed(k)) {
          k.levetid = 0;
          return true;
        }
      }
      
      noFill();
      stroke(255);
      
      translate(x, y);
      rotate(vinkel);
      
      beginShape();
      for (int i = 0; i < punkterX.length; i++) {
        vertex(punkterX[i], punkterY[i]); // Bare denne linjen er endret
      }
      endShape(CLOSE);
      
      resetMatrix();
      
      return false;
    }
    ...
  }
  ```
  
  Så må vi endre på løkken som tegner opp asteroidene, sånn at vi kan fjerne asteroiden:
  
  ```processing
  void draw() {
    oppdaterSkip();
    
    background(0);
    
    translate(x, y);
    rotate(vinkel);
  
    noStroke();
    fill(255);
    triangle(-10, -10, -10, 10, 20, 0);
    
    if (fram) {
      stroke(255);
      noFill();
      ellipse(-15, 0, 10, 10);
      line(-20, 0, -25, 0);
    }
    
    resetMatrix();
    
    // Tegn kulene
    for (Kule kule : kuler) {
      kule.draw();
    }
    
    // Tegn asteroidene
    Iterator<Asteroide> it = asteroider.iterator();
    while (it.hasNext()) {
      Asteroide ast = it.next();
      if (ast.draw()) {
        it.remove();
      }
    }
  }
  ```
  
  Hvis du kjører spillet nå, vil du se at asteroiden og kulen forsvinner når du skyter en asteroide. Her har vi et par ting som du kanskje ikke har sett før: `for`-løkken er byttet ut med en `while`-løkke og vi har tatt i bruk iteratoren. Begge disse er derfor forklart i boksene under.
  
### While {.protip}

Denne typen løkker, er enda litt enklere enn `for`-løkker, i hvert fall på noen måter. Akkurat som den enkle `for`-løkken, gjør den en sjekk for om den skal kjøre, eller om den er ferdig. Denne sjekken kalles en betingelse og løkken blir noen ganger kalt en betingelsesløkke. I motsetning til en `for`-løkke har den ikke en initieringssetning eller en oppdateringssetning som en del av strukturen. Derfor kan `for`-løkken sees på som en utvidelse av `while`-løkken, der den ene setningen settes foran løkken og den andre til slutt i løkken. Det er ikke en helt nøyaktig tilnærming, men passer ganske godt. Siden det ikke finnes en initieringssetning, må all forberedelse gjøres før løkken. Og siden det ikke finnes en oppdateringssetning, må man passe på at tilstanden endrer seg inne i løkken for at løkken skal kunne ta slutt. De to løkkene under gjør det samme, men i noen tilfeller egner den ene seg bedre enn den andre.

```processing
for (int i = 0; i < 10; i++) {
  sum += i;
}
```

```processing
int i = 0;
while (i < 10) {
  sum += 10;
  i++;
}
```

Et annet eksempel er som det vi hadde tidligere. De to følgende løkkene er like, men om du skal fjerne elementer i listen må du bruke en `while`- eller vanlig `for`-løkke (ikke en `for-each` som under, men en `for` som den i forrige eksempel).

```processing
for (Type t : liste) {
  t.gjoerNoe();
}
```

```processing
Iterator<Type> it = liste.iterator();
while (it.hasNext()) {
  Type t = it.next();
  t.gjoerNoe();
}
```

Som du ser må du ofte skrive en eller to setninger ekstra for å bruke en `while`-løkke, men av og til er det enklere enn å bruke `for`. For eksempel løkken under:

```processing
Iterator<Type> it = liste.iterator();
while (it.hasNext()) {
  Type t = it.next();
  if (t.skalSlettes()) {
    it.remove();
  }
}
```

```processing
for (Iterator<Type> it = liste.iterator(); it.hasNext();) {
  Type t = it.next();
  if (t.skalSlettes()) {
    it.remove();
  }
}
```

Her er forskjellen mellom de to løkkene ganske liten, men man bruker en litt spesiell egenskap ved `for`-løkken: en eller flere av setningene i parentesene kan stå tom. Her har oppdateringssetningen blitt utelatt, for `it.next()` må kalles i begynnelsen av løkken, og ikke i slutten.

### Iterator {.protip}

En iterator lar deg løpe gjennom elementene i en datastruktur, som f.eks. en liste. Den har tre metoder du kan kalle: `hasNext`, `next` og `remove`. `hasNext` returnerer `true` hvis `next` kommer til å returnere noe. Hvis `hasNext` returnerer `false` vil det skje en feil om man kaller `next`. `next` henter ut det neste elementet i datastrukturen og oppdaterer noe tilstand inne i iteratoren slik at neste kall på `hasNext` og `next` vil fungere som forventet. Altså, et kall på `next` henter neste element fra datastrukturen, og flytter seg et hakk fram i datastrukturen, slik at hvis det er flere elementer igjen vil neste kall på `next` få ut dette. Dette gjør at disse to metodene sammen fungerer bra for å lage en `while`-løkke som løper gjennom en liste eller annen datastruktur. Den siste metoden `remove`, fjerner det elementet som sist ble hentet med `next`, fra datastrukturen. Det går ikke å kalle `remove` uten at `next` har blitt kalt siden forrige `remove` eller siden opprettelsen av iteratoren.

Du lurer kanskje på hvorfor man ikke kan legge til elementer med en iterator når man kan fjerne elementer. Faktisk finnes det noen typer iteratorer som tillater at man legger til elementer i datastrukturen også, eller at man beveger seg gjennom datastrukturen i en annen rekkefølge. Disse dekkes ikke her, men om du er interessert i dette kan du ta en titt på f.eks. **TODO Link this** `ListIterator`.

##

+ Nå må vi legge til nye asteroider hvis asteroiden som ble ødelagt er stor nok. For dette skal vi bruke en ekstra liste med asteroider. Legg følgende til før `setup`:
  
  ```processing
  ArrayList<Aseroide> nyeAsteroider = new ArrayList<Asteroide>();
  ```

+ Når en asteroide eksploderer, så legger vi til to nye asteroider hvis asteroiden var stor eller middels stor:
  
  ```processing
  void draw() {
    oppdaterSkip();
    
    background(0);
    
    translate(x, y);
    rotate(vinkel);
  
    noStroke();
    fill(255);
    triangle(-10, -10, -10, 10, 20, 0);
    
    if (fram) {
      stroke(255);
      noFill();
      ellipse(-15, 0, 10, 10);
      line(-20, 0, -25, 0);
    }
    
    resetMatrix();
    
    // Tegn kulene
    for (Kule kule : kuler) {
      kule.draw();
    }
    
    // Tegn asteroidene
    Iterator<Asteroide> it = asteroider.iterator();
    while (it.hasNext()) {
      Asteroide ast = it.next();
      if (ast.draw()) {
        it.remove();
        
        if (ast.radius >= 10) {
          float retning = random(0, 2 * PI);
          float fart = random(MAKS_FART);
          Asteroide nyAsteroide = new Asteroide(ast.x, ast.y, fart * cos(retning), fart * sin(retning), ast.radius / 2.0);
          nyAsteroide.draw();
          nyeAsteroider.add(nyAsteroide);
          
          retning = retning + PI;
          nyAsteroide = new Asteroide(ast.x, ast.y, fart * cos(retning), fart * sin(retning), ast.radius / 2.0);
          nyAsteroide.draw();
          nyeAsteroider.add(nyAsteroide);
        }
      }
    }
    
    if (!nyeAsteroider.isEmpty()) {
      asteroider.addAll(nyeAsteroider);
      nyeAsteroider.clear();
    }
  }
  ```
  
  Så etter at vi har fjernet den nettopp sprengte asteroiden, sjekker vi om den er stor nok og hvis den er det så lager vi to nye asteroider som flyr i motsatt retning av hverandre i samme hastighet. Legg merke til at vi tegner opp de to nye asteroidene også her, da kan det skje en litt uventet ting. En kule kan i teorien treffe en av disse asteroidene. Dette har vi ikke tatt hensyn til her, men det kunne vi gjort ved å flytte koden som lager nye asteroider inn i `Asteroide.draw`, men da måtte vi sendt med listen `nyeAsteroider`. Hvis en kule treffer en av de nye asteroidene, som er nesten umulig pga. avstanden mellom kuler og hastighetene asteroidene kan bevege seg i, vil kulen forsvinne, og den nye asteroiden først dukke opp neste opptegning. Legg merke til at vi gjenbruker variablene `retning`, `fart` og `nyAsteroide` når vi skal opprette asteroide nummer to.
  
  Etter løkken som tegner opp asteroidene flytter vi alle de nye asteroidene til lista `asteroider`. Og så tømmer vi lista med de nye asteroidene. Dette gjør vi bare om det finnes noen nye asteroider.

## Game Over {.check}
  
  Asteroidene skal jo ikke bare ødelegges av kulene, men ødelegge skipet til spilleren om spilleren kolliderer med dem. 

# Finpussen {.activity}

## Nytt spill {.check}

Når spilleren taper, må de få lov å starte et nytt spill. Det hadde også vært fint om spillet ikke begynte med en gang, men ventet til spilleren ba om det. Så la oss få på plass en start-skjerm som vises til spilleren er klar.

## Flere brett {.check}

Det er litt kjedelig når man ikke kommer videre etter at man har skutt i stykker alle asteroidene. Vi har allerede fått på plass at spillet ikke starter når `setup` blir kjørt, så la oss bygge videre på dette sånn at vi kan starte nye brett når ett er over.

## Rekordtavle {.check}

Nå som man kan spille og samle opp masse poeng, er det litt trist at ingen kan se hvor god man har vært etterpå. Derfor skal vi lagre de ti beste resultatene og vise dem til spillerne. Hvis man får nok poeng til å være blant de ti beste, får man skrive inn navnet sitt som så vises på rekordtavlen.

# Videreutvikling {.activity}

Nå er spillet ganske komplett, men det finnes alltid rom for å utvide ting og her er noen ideer til hva du kan gjøre videre om du har lyst til å gjøre mer med spillet.

+ Fiender
  
  Noen steiner som flyter rundt i verdensrommet kan ikke akkurat kalles fiender. De er hindre som man må overkomme, men vi kan gjøre enda mer. Ataris klassiker *Asteroids* hadde romskip (flyvende tallerkener) som kom flyvende over skjermen og skøyt etter spilleren. Hvis man skøyt ned disse, fikk man masse bonuspoeng. De kom i to utgaver, en stor og en liten. Den store var ganske enkel å skyte ned og siktet dårlig. Den lille var raskere, vanskeligere å treffe fordi den var liten, og siktet mer nøyaktig mot der spilleren kom til å være om de ikke skiftet kurs.
  
  Du trenger ikke å lage akkurat de samme typene romskip som fantes i originalen, men du kan legge til fiender som prøver å angripe spilleren som man får mer poeng for å skyte ned enn asteroidene.

+ Grafikk
  
  Her har vi vært ganske tro mot originalen, og lagd et spill som ser nesten helt likt ut. Grafikken er derfor ganske enkel. Trekanter, streker og mangekanter. Det ville sett mer imponerende ut med noe litt stiligere. Du kan tegne bilder som du kan bruke istedenfor, eller lage mer detaljerte tegninger ved å bruke farger og flere mangekanter. Det finnes utrolig mange muligheter for hva du kan gjøre med grafikken. Du kan også legge til ting som eksplosjoner, stjerner og andre detaljer.

+ Flere spillere
  
  Det er morsommere å spille sammen. Kan du gjøre om spillet til å fungere for to spillere? Hvordan får du best til at det kan være flere skip samtidig? Husk at du må oppdatere styringen også.

