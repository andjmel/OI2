# 📊 Upravljanje zalihama — Excel Cheatsheet

Konkretne formule iz fajlova, grupisane po temama. Formule su prikazane generički (npr. red 6) — kopiraj i prilagodi broj reda.

---

## 1. ZADATAK — Optimizacija (sheet: `ZADATAK - UZ`)

### Q* — Optimalna količina narudžbine (sa penalima)
```excel
=SQRT((2*(C5+C7+C9)*J5)/(2*J4)) * SQRT((C8+C10)/(C10))
```
> `C5`=Cs(fix), `C7`=Cs(fix2?), `C9`=Ch(fix), `J5`=N (tražnja), `J4`=T (period), `C8`=Ch(var), `C10`=Cp(var)

### Q* — bez penala (Zadatak II, jednostavniji model)
```excel
=SQRT((2*(C5+C7)*J5)/(3*J4))
```

### y — Nivo zaliha pri isporuci
```excel
=SQRT((2*(C5+C7+C9)*J5)/(2*J4)) * SQRT(C10/(C8+C10))
```

### n — Broj narudžbina
```excel
=J5/C15
```
> `J5`=N, `C15`=Q*

### θ — Dužina ciklusa (dani)
```excel
=C18*(C16/C15)
```
> `C18`=θ (dužina ciklusa), `C16`=y, `C15`=Q*  
> Ili direktno: `=J4/C17` (period / broj narudžbina)

### θ1 i θ2
```excel
=C18*(C16/C15)      ← θ1 (vreme sa zalihama)
=C18*((C15-C16)/C15) ← θ2 (vreme čekanja/deficita)
```

### Pomoćne vrednosti
```excel
=2*30              ← Ch(var)/2 ili slično (hardkodovano)
=0.5*20            ← Cp(var)/2
=80*30             ← N = d * T
```

---

## 2. PERIODIČNO PRAĆENJE ZALIHA — PP

### Kolona C — Odbrojavanje do revizije / ulaza
```excel
=IF(C4, C4-1, $B$4-1)
```
> Odbrojava od PR-1 do 0. Kad dođe do 0 → revizija ili ulaz.

### Kolona D — Oznaka "Rev." ili "Ul."
```excel
=IF(C5, IF(C5=$B$4-$B$5, $A$7, ""), $A$8)
```
> Ako je odbrojavanje na vrednosti `PR - D` → označi "Ul." (`$A$7`).  
> Ako je odbrojavanje na 0 → označi "Rev." (`$A$8`).  
> `$B$4`=PR (period revizije), `$B$5`=D (vreme isporuke)

### Kolona F — Ulaz robe (stiže narudžbina)
```excel
=IF(D6=$A$7, I5, 0)
```
> Ako je danas dan ulaza ("Ul.") → uzmi narudžbinu iz prethodnog reda (I5), inače 0.

### Kolona G — Slučajni izlaz (tražnja)
```excel
=IF(RAND()>0.2, INT(RAND()*15), 0)
```
> Simulira dnevnu tražnju (80% šanse da bude između 0-14, 20% šanse da bude 0).

### Kolona H — Stanje zaliha
```excel
=MAX(H5 - G6 + F6, 0)
```
> Prethodne zalihe − izlaz + ulaz. MAX sa 0 sprečava negativne zalihe (u osnovnom modelu).

**Varijanta sa karantinom (Zadatak II):**
```excel
=IF(D6=$A$8, MAX(H5-G6+F6+L5, 0), MAX(H5-G6+F6-L5, 0))
```
> `L5` = karantin (odložena tražnja koja se dodaje/oduzima zavisno od dana)

### Kolona I — Narudžbina
```excel
=IF(D6=$A$8, $B$6-H6, IF(D6=$A$7, 0, I5))
```
> Ako je danas revizija ("Rev.") → naruči do NR: `NR - H6`.  
> Ako je dan ulaza → narudžbina postaje 0 (roba stigla).  
> Inače → zadrži prethodnu narudžbinu.

**Varijanta sa minimalnom količinom (Zadatak I):**
```excel
=IF(D6=$A$8, IF(($B$6-H6)>0, MAX($B$6-H6, 200), 0), IF(D6=$A$7, 0, I5))
```
> Narudžbina ne može biti manja od 200 paketa.

### Kolona J — "5+6" (zalihe + narudžbina u čekanju)
```excel
=H6 + I6
```

### Kolona K — Provera (stvarno stanje bez MAX)
```excel
=H5 - G6 + F6
```
> Negativna vrednost ovde = deficit (ako je H kolona zakočena na 0).

### Kolona L — Karantin (Zadatak II)
```excel
=IF(D6=$A$8, 0, IF(MOD(K6, 3)=0, ROUND(H6*0.15, 0), L5))
```
> Svakog 3. dana (MOD) 15% zaliha ide u karantin.

### Sumarni redovi
```excel
=SUM(F5:F49)           ← ukupni ulazi
=SUM(G5:G49)           ← ukupni izlazi
=COUNTIF(D5:D49,"Rev.")  ← broj revizija
=AVERAGE(H5:H49)       ← prosečne zalihe
```

---

## 3. STALNO PRAĆENJE ZALIHA — SP

### Kolona C — Odbrojavanje do ulaza
```excel
=IF(D5=$A$6, $B$3-1, IF(C5, C5-1, 0))
```
> Ako je juče data narudžbina (`$A$6` = oznaka narudžbine) → počni odbrojavanje od D-1.  
> Inače odbrojavaj do 0.

### Kolona D — Narudžbina (okidač)
```excel
=IF(AND(I6<$B$5, NOT(C6)), $A$6, "")
```
> Ako su zalihe ispod Pc (`$B$5`) I nije u toku isporuka (C6=0) → daj narudžbinu.

### Kolona E — Oznaka ulaza
```excel
=IF(C6, "", IF(C5, $A$6, ""))
```
> Ako odbrojavanje dostiže 0 (C5 je bio >0, C6 je 0) → danas stiže roba.

### Kolona G — Ulaz robe
```excel
=IF(E6=$A$6, J5, 0)
```
> Ako je danas dan ulaza → primi naručenu količinu Q (J5 = narudžbina iz prethodnog reda).

### Kolona H — Dnevna tražnja (slučajna)
```excel
=IF(RAND()>0.2, INT(RAND()*$B$2), 0)
```
> `$B$2` = maksimalna dnevna tražnja.

### Kolona I — Stanje zaliha
```excel
=MAX(I5 - H6 + G6, 0)
```
> Prethodne zalihe − tražnja + ulaz.

**Varijanta sa periodičnom inspekcijom (Zadatak I, svakih 10 dana):**
```excel
=IF(MOD(L6,10)=0, MAX((I5-INT(I5*0.1))+G6-H6, 0), MAX(I5-H6+G6, 0))
```
> Svakih 10 dana ukloni 10% zaliha (inspekcija/otpis).

**Varijanta sa periodičnim otpisom (Zadatak II, svakih 8 dana):**
```excel
=MAX(IF(MOD(L6,8)=0, I5-2*80-H6+G6, I5-H6+G6), 0)
```

### Kolona J — Narudžbina u čekanju
```excel
=IF(D6=$A$6, $B$4, IF(E6=$A$6, 0, J5))
```
> Ako danas naručujemo → stavi Q (`$B$4`).  
> Ako danas stiže roba → postavi na 0.  
> Inače → zadrži prethodnu vrednost.

### Kolona K — "5+6"
```excel
=I5 + J5
```

### Sumarni redovi
```excel
=SUM(G5:G64)             ← ukupni ulazi
=SUM(H5:H64)             ← ukupni izlazi/tražnja
=COUNTIF(D5:D65,"=1")    ← broj narudžbina
=AVERAGE(I5:I64)         ← prosečne zalihe
```

---

## 4. TROŠKOVI (sheet: `TROŠKOVI`)

### Troškovi naručivanja (PP)
```excel
=COUNTIF('Periodično praćenje zaliha (PP)'!F5:F64,">0")*C5 + AVERAGE('Periodično praćenje zaliha (PP)'!H5:H64)*C6
```
> Broj ulaza × Cs(fix) + prosečne zalihe × Cs(var)

### Troškovi držanja (PP)
```excel
=COUNTIF('Periodično praćenje zaliha (PP)'!F5:F64,">0")*C7 + AVERAGE('Periodično praćenje zaliha (PP)'!H5:H64)*C8*60
```
> Broj ulaza × Ch(fix) + prosečne zalihe × Ch(var) × T

### Troškovi penala (PP)
```excel
=COUNTIF('Periodično praćenje zaliha (PP)'!F5:F64,">0")*C9 + ABS(AVERAGEIF('Periodično praćenje zaliha (PP)'!K6:K64,"<0"))*C10*60
```
> Broj ulaza × Cp(fix) + prosečan dnevni deficit × Cp(var) × T  
> `AVERAGEIF(...,"<0")` hvata samo negativne vrednosti iz kolone K (stvarni deficiti)

### Troškovi dolaska / revizije (PP)
```excel
=COUNTIF('Periodično praćenje zaliha (PP)'!D5:D64,"=Rev.")*C16
```
> Broj revizija × cena po dolasku

### Troškovi naručivanja (SP)
```excel
=COUNTIF('Stalno praćenje zaliha (SP)'!G5:G64,">0")*C5 + AVERAGE('Stalno praćenje zaliha (SP)'!I5:I64)*C6
```

### Troškovi držanja (SP)
```excel
=COUNTIF('Stalno praćenje zaliha (SP)'!G5:G64,">0")*C7 + AVERAGE('Stalno praćenje zaliha (SP)'!I5:I64)*C8*60
```

### Troškovi penala (SP)
```excel
=COUNTIF('Stalno praćenje zaliha (SP)'!G5:G64,"=0")*C9 + ABS(AVERAGEIF('Stalno praćenje zaliha (SP)'!M6:M64,"<0"))*C10*60
```
> `G="=0"` → dani bez ulaza (nema robe) = penalizovani dani

### Troškovi tehničkog pregleda (SP)
```excel
=C21*60
```
> Dnevni trošak tehničara × broj dana

### Troškovi raspona (Zadatak II — varijanta)
```excel
=(MAX('Periodično praćenje zaliha (PP)'!H5:H34) - MIN('Periodično praćenje zaliha (PP)'!H5:H34)) * C14 * 'ZADATAK - UZ'!J4
```

### Ukupni troškovi
```excel
=SUM(G5:G8)     ← PP ukupno
=SUM(G13:G19)   ← SP ukupno
```

---

## 5. BRZA REFERENCA — Šta ide u koju kolonu

| Kolona | PP model | SP model |
|--------|----------|----------|
| B/C | Parametri (PR, D, NR, Q, Pc) | Parametri |
| C (u tabeli) | Odbrojavanje do Rev/Ul | Odbrojavanje do ulaza |
| D | Oznaka "Rev." ili "Ul." | Oznaka narudžbine (1) |
| E | — | Oznaka ulaza |
| F | Ulaz robe | — |
| G | Slučajni izlaz (tražnja) | Ulaz robe |
| H | Stanje zaliha | Slučajni izlaz (tražnja) |
| I | Narudžbina u čekanju | Stanje zaliha |
| J | Zalihe + narudžbina (5+6) | Narudžbina u čekanju |
| K | Provera (bez MAX) | Zalihe + narudžbina (5+6) |
| L | Karantin (opciono) | Brojač dana (za MOD) |

---

*Izvučeno iz: `UPRAVLJANJE_ZALIHAMA_osnovni_okvir.xlsx`, `_I_zadatak.xlsx`, `_II_zadatak.xlsx`*
