### @activities 1

## Battlebot - programmering av mottakerprogram

### Programmer mottakerprogram til enkel fjernkontroll @unplugged
Vi skal nå programmere mottakerprogrammet som sammen med avansert fjernkontroll styrer battleboten vår.   
Det er en fordel om dere allerede har laget programmet "avansert fjernkontroll" for vi skal i dette programmet bestemme hva battleboten gjør når den mottar de ulike signalene fra fjernkontrollen.

Vi må huske hvilke verdier vi har programmert fjernkontrollen til å sende. (Tallene under er standardverdiene i tutorialen)     
__Når ristes__ - skrur battleboten _på_   - sender __A__ og tallet __1__ for _på_ eller __0__ for _av_ via radio   
__Pitch__ - styrer battleboten _hastigheten_  - sender __H__ og __verdien__ for hvor mye vi tipper microbit frem eller tilbake via radio.   
__Roll__ - styrer om battleboten svinger til _høyre_ eller _venstre_  - sender __S__ og __verdien__ for hvor mye vi heller microbit mot høyre eller venstre via radio   
   

## Motta signaler

### Steg 1
Når microbit mottar radiosignaler fra fjernkontrollen må vi tolke disse. Her må vi bruke _vilkår_ og undersøke både navnet og verdien som vi får via radio.  
Hent ``||radio:Når radio mottar <name> <value>||`` fra kategorien radio. Hent ``||logic:Hvis ellers||`` fra logikk og plasser den i ``||radio:Når radio mottar <name> <value>||``   
Legg inn blokker som sjekker om navnet er H, A eller S.
```blocks
radio.onReceivedValue(function (name, value) {
    if (name == "H") {
    }
    if (name == "A") {
    }
    if (name == "S") {
    }
})
```

### Steg 2

Lag tre variabler, ``||variables:Kjør||``, ``||variables:AvPå_Bil||`` og ``||variables:Justering||`` som dere bruker for å oppbevare verdiene som mottas via radio.  
Legg inn i ``||logic:Hvis ellers||`` blokken at: 

Hvis vi mottar H - sett ``||variables:Kjør||`` til _value_  
Hvis vi mottar A - sett ``||variables:AvPåBil||`` til _value_    
Hvis vi mottar S - sett ``||variables:Justering||`` til _value_

```blocks
radio.onReceivedValue(function (name, value) {
    if (name == "H") {
        Kjør = value
    }
    if (name == "A") {
        AvPå_Bil = value
    }
    if (name == "S") {
        Justering = value
    }
})
```
### Steg 3

Vi skal spare oss for litt jobb og gjør en endring i Kjør = value.  
Verdien vi får fra fjernkontrollen vil være mellom -90 og 90, der -90 vil være det vi tenker på som full fart fremover. Vi vil derfor regne om disse verdiene slik at full fart fremover fra fjernkontrollen blir full fart fremover til motorene. Vi kan styre farten på motorene med verdier fra 0 til 255. 
Vi bruker derfor ``||math:regn om||`` og regner om verdien til value fra -90, 90 til -200, 200.

```blocks
radio.onReceivedValue(function (name, value) {
    if (name == "H") {
        Kjør = Math.map(value, -90, 90, -200, 200)
    }
    if (name == "A") {
        AvPå_Bil = value
    }
    if (name == "S") {
        Justering = value
    }
})
```

### Steg 3
Sett ``||radio:radiogruppe||`` og lag variablene ``||variables:Stopp||``, ``||variables:venstrejustering||`` og ``||variables:høyrejustering||`` i ved start.  
Sett stopp til 0 og radiogruppe til samme tall som dere brukte når dere programmerte fjernkontrollen.

```blocks
radio.setGroup(xx)
let Stopp = 0
let Høyrejustering = 0
let Venstrejustering = 0
```

### Steg 5
I gjenta for alltid vil dere at battleboten skal undersøke om den er på eller ikke. 
Bruk en ``||logic:hvis ellers||`` blokk. 
Hvis ``||variables:AvPå_Bil = 1||`` skal Battleboten kjøre, ellers skal den stoppe. 
Bruk blokkene fra DF-Driver for å stoppe motorene. ``||DF-Driver:Motor M1 dir CW speed||``

```blocks
basic.forever(function () {
    if (AvPå_Bil) {
    }    else {
        motor.MotorRun(motor.Motors.M1, motor.Dir.CCW, Stopp)
        motor.MotorRun(motor.Motors.M2, motor.Dir.CCW, Stopp)
    }
}
```

### Steg 6

For å få battleboten til å kjøre frem eller tilbake bruker vi variablen ``||variables:Kjør||`` som inneholder verdien for _hastighet_ som vi mottar fra fjernkontrollen.
Verdien til Kjør kan være et sted mellom -200 og 200 der -200 vil være det vi tenker på som full fart fremover på fjernkontrollen. Farten til motorene kan derimot bare være et positivt tall. 
Vi bruker derfor vilkår og undersøker om verdien til kjør er større eller lik 0. Som tilsvarer at fjernkontrollen tippes mot oss. Da skal battleboten gå bakover. 
Hent to blokker av ``||DF-Driver:Motor M1 dir CW speed||`` fra kategorien DF-Driver. Plasser begge blokkene under hvis Kjør >= 0. Endre den ene blokken slik at den sier ``||DF-Driver:Motor M2 dir CW speed||``
Her må dere velge rotasjon som stemmer med motorene slik de er montert på deres battlebot. CW står for ClockWise og CCW CounterClockWise.   
Motorene kan være montert slik at de går i motsatt retning selv om det i koden står CW på både motor 1 og 2. Gjør nødvendige endringer i programmet slik at det blir riktig for dere.
Her kan det være at dere trenger å teste programmet noen ganger før det blir riktig.

```blocks
basic.forever(function () {
    if (AvPå_Bil) {
        if (Kjør >= 0) {
            motor.MotorRun(motor.Motors.M1, motor.Dir.CCW, Kjør)
            motor.MotorRun(motor.Motors.M2, motor.Dir.CW, Kjør)
        } else {
      }
    } else {
        motor.MotorRun(motor.Motors.M1, motor.Dir.CCW, Stopp)
        motor.MotorRun(motor.Motors.M2, motor.Dir.CCW, Stopp)
    }
})
```

### Steg 6
Hvis ``||variables:Kjør < 0||`` så vil dette bety at vi med fjernkontrollen vil at battleboten skal gå fremover. Vi kan da bruke ``|math:absoluttverdi||`` blokken og legge inn absoluttverdien av Kjør som farten til motorene når Kjør < 0.
 
```blocks
basic.forever(function () {
    if (AvPå_Bil) {
        if (Kjør >= 0) {
            motor.MotorRun(motor.Motors.M1, motor.Dir.CCW, Kjør)
            motor.MotorRun(motor.Motors.M2, motor.Dir.CW, Kjør)
        } else {
            motor.MotorRun(motor.Motors.M1, motor.Dir.CW, Math.abs(Kjør))
            motor.MotorRun(motor.Motors.M2, motor.Dir.CCW, Math.abs(Kjør))
        }
    } else {
        motor.MotorRun(motor.Motors.M1, motor.Dir.CCW, Stopp)
        motor.MotorRun(motor.Motors.M2, motor.Dir.CCW, Stopp)
    }
})
```
### Steg 7

Det er flere mulige måter å legge inn sving på. Her kommer en mulighet. Kanskje finner dere en bedre måte å gjøre det på?
Vi har allerede lagd en variabel  for høyrejustering og en variabel for venstrejustering. Disse skal vi bruke for å justere farten på motorene slik at battleboten svinger.
Når battlebot skal svinge mot venstre, må venstre motor gå saktere og motsatt når man svinger mot høyre.
Vi kan sette venstrejustering = Kjør - (regn om _Justering_ fra lav 0 til høy - 90 til lav Kjør og høy Kjør * -1)  
På den måten blir justeringen av svingen ikke større enn den farten som vi sier battleboten skal kjøre i.
Vi gjøre det samme for høyrejustering. 
Farten til motorene vil da for venstre motor (M1) blir (``||variables:Kjør||`` - ``||variables:Venstrejustering||``) og tilsvarende med høyrejustering på høyre motor.


```blocks
basic.forever(function () {
    if (AvPå_Bil) {
        if (Kjør >= 0) {
            Venstrejustering = Kjør - Math.map(Justering, 0, -90, Kjør, Kjør * -1)
            Høyrejustering = Kjør - Math.map(Justering, 0, 90, Kjør, Kjør * -1)
            motor.MotorRun(motor.Motors.M1, motor.Dir.CCW, Kjør - Venstrejustering)
            motor.MotorRun(motor.Motors.M2, motor.Dir.CCW, Kjør - Høyrejustering)
        } else {
            Venstrejustering = Kjør + Math.map(Justering, 0, -90, Kjør, Kjør * -1)
            Høyrejustering = Kjør + Math.map(Justering, 0, 90, Kjør, Kjør * -1)
            motor.MotorRun(motor.Motors.M1, motor.Dir.CW, Math.abs(Kjør) - Venstrejustering)
            motor.MotorRun(motor.Motors.M2, motor.Dir.CW, Math.abs(Kjør) - Høyrejustering)
        }
    } else {
        motor.MotorRun(motor.Motors.M1, motor.Dir.CCW, Stopp)
        motor.MotorRun(motor.Motors.M2, motor.Dir.CCW, Stopp)
    }
})
```
### Steg 8
Dere er nå klare for Battle - eller kanskje dere ønsker å legge til noen servomotorer til å styre ulike våpen? Da finner dere blokkene som styrer servo i DF-Driver-kategorien. 
Lykke til!


```package
DF-Driver=github:DFRobot/pxt-motor
``` 
