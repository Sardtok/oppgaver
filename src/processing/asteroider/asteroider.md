---
title: Asteroider
level: 2
author: Sigmund Hansen
---

# Introduksjon {.intro}

Dette er første del av en serie oppgaver hvor vi skal utvikle spillet Asteroider, basert på Atari-klassikeren Asteroids. I dette spillet flyr man et romskip rundt på skjermen og skyter ned asteroider. Når man skyter en asteroide, brytes den opp i flere mindre asteroider. De minste asteroidene går helt i stykker når man skyter dem. Om man blir truffet av en asteroide, dør man.

Først skal vi lage et romskip som man kan fly rundt på skjermen, og når man flyr ut av skjermen på en side, dukker man opp på motsatt side av skjermen. Her vil vi ha fokus på opptegning av romskipet, håndtering av tastetrykk og å flytte skipet rundt på skjermen.

Så skal vi legge til asteroider som svever rundt i rommet. Disse skal ødelegge romskipet om de kolliderer med det, og gå i stykker om man skyter dem. Når de går i stykker, deles de opp i mindre deler som kastes ut i rommet. Disse må man også passe seg for og skyte ned.

Til slutt skal vi legge en avsluttende hånd på verket. Poeng og liv skal vises, man skal få ekstraliv når man når bestemte poengsummer, og man skal ha en startskjerm som man sendes tilbake til når man taper, sånn at man kan begynne på nytt.

Det vil fortsatt være en del rom for forbedringer som man kan legge til. Lagring av rekorder, UFOer som skyter på spilleren som i klassikeren og å bruke bildefiler til grafikken isteden.

# Skipet - Opptegning

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

# Skipet - Styring

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

## Snu skipet

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

## Fly

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

