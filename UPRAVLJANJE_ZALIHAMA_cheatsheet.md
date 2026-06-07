# 📦 Upravljanje zalihama — Cheatsheet formula

> Sve ključne formule za optimizaciju zaliha, praćenje, troškove i analizu.

---

## 1. Osnovni parametri (oznake)

| Oznaka | Značenje | Jedinica mere |
|---|---|---|
| `T` | Planski period | dani |
| `N` | Ukupna tražnja u periodu T | paketa |
| `d` | Dnevna tražnja: `d = N / T` | paketa/dan |
| `Q` | Količina jedne narudžbine | paketa |
| `Q*` | Optimalna količina narudžbine (EOQ) | paketa |
| `y` | Nivo zaliha pri isporuci (max zalihe nakon ulaza) | paketa |
| `n` | Broj narudžbina u periodu | narudžbina |
| `θ` | Dužina jednog ciklusa narudžbine | dani |
| `θ1` | Vreme od ulaza do pada zaliha na nulu | dani |
| `θ2` | Vreme čekanja (deficit, penalizacija) | dani |

---

## 2. Troškovi — oznake i opis

| Oznaka | Opis | Jedinica mere |
|---|---|---|
| `Cs(fix)` | Fiksni trošak naručivanja (po narudžbini, bez obzira na količinu) | n.j. |
| `Cs(var)` | Varijabilni trošak naručivanja (po paketu koji se naručuje) | n.j./paket |
| `Ch(fix)` | Fiksni trošak držanja zaliha (po ciklusu) | n.j. |
| `Ch(var)` | Varijabilni trošak držanja (po paketu po danu) | n.j./paket/dan |
| `Cp(fix)` | Fiksni trošak penala/čekanja (po ciklusu deficita) | n.j. |
| `Cp(var)` | Varijabilni trošak penala (po paketu po danu čekanja) | n.j./paket/dan |

---

## 3. Optimizacija (EOQ model sa penalima)

### 3.1 Optimalna količina narudžbine — `Q*`

$$Q^* = \sqrt{\frac{2 \cdot N \cdot (Cs_{fix} + Cs_{var} \cdot Q^*)}{Ch_{var}} \cdot \frac{Cp_{var} + Ch_{var}}{Cp_{var}}}$$

**Praktična aproksimacija (bez penala, klasični EOQ):**

$$Q^* = \sqrt{\frac{2 \cdot N \cdot Cs_{fix}}{Ch_{var}}}$$

**Sa penalima (puna formula):**

$$Q^* = \sqrt{\frac{2 \cdot N \cdot Cs_{total}}{Ch_{var}} \cdot \frac{Ch_{var} + Cp_{var}}{Cp_{var}}}$$

> gde je `Cs_total = Cs(fix) + Cs(var) · (N/T)`

---

### 3.2 Optimalni nivo zaliha pri isporuci — `y`

$$y = Q^* \cdot \frac{Cp_{var}}{Ch_{var} + Cp_{var}}$$

> `y` je maksimalni nivo zaliha odmah posle ulaza robe. Uvek je `y ≤ Q*`.

---

### 3.3 Broj narudžbina — `n`

$$n = \frac{N}{Q^*}$$

---

### 3.4 Trajanje ciklusa — `θ`

$$\theta = \frac{T}{n} = \frac{Q^*}{d}$$

---

### 3.5 Vreme sa zalihama (pozitivne zalihe) — `θ1`

$$\theta_1 = \frac{y}{d}$$

---

### 3.6 Vreme čekanja / deficita — `θ2`

$$\theta_2 = \theta - \theta_1 = \frac{Q^* - y}{d}$$

---

## 4. Ukupni troškovi

### 4.1 Troškovi naručivanja (po periodu T)

$$TC_s = n \cdot Cs_{fix} + N \cdot Cs_{var}$$

> Fiksni deo se množi brojem narudžbina, varijabilni ukupnom količinom.

---

### 4.2 Troškovi držanja zaliha (po periodu T)

$$TC_h = n \cdot Ch_{fix} + Ch_{var} \cdot \frac{y \cdot \theta_1}{2}$$

> Prosečne zalihe u toku θ1 su `y/2`, pa je površina trougla `y · θ1 / 2`.

---

### 4.3 Troškovi penala / čekanja (po periodu T)

$$TC_p = n \cdot Cp_{fix} + Cp_{var} \cdot \frac{(Q^* - y) \cdot \theta_2}{2}$$

> Prosečni deficit u toku θ2 je `(Q* - y) / 2`.

---

### 4.4 Ukupni trošak (zbir)

$$TC_{ukupno} = TC_s + TC_h + TC_p$$

---

## 5. Periodično praćenje zaliha (PP)

U modelu **periodičnog praćenja** zalihe se pregledaju svakih `PR` dana (period revizije). Narudžbina se daje samo u momentu revizije.

### 5.1 Ključne oznake PP

| Oznaka | Opis |
|---|---|
| `PR` | Period revizije (broj dana između dva pregleda) |
| `D` | Vreme isporuke (dana od narudžbine do ulaza robe) |
| `NR` | Nivo popunjavanja — ciljni nivo do kojeg se naručuje |

### 5.2 Količina narudžbine u PP

$$Q_{nar} = NR - Z_{trenutne}$$

> Gde je `Z_trenutne` stanje zaliha u momentu revizije.

### 5.3 Stanje zaliha (kolona u tabeli)

$$Z_t = Z_{t-1} + Ulaz_t - Izlaz_t$$

### 5.4 Kolona „5+6" (raspoložive + naručene zalihe)

$$[5+6]_t = Z_t + Narudžbina_t$$

> Govori koliko robe imamo ili očekujemo — korisno za procenu budućih nedostataka.

### 5.5 Prosečne zalihe (PP)

$$\bar{Z}_{PP} = \frac{\sum Z_t}{T}$$

---

## 6. Stalno praćenje zaliha (SP)

U modelu **stalnog praćenja** zalihe se prate svaki dan. Narudžbina se daje čim zalihe padnu ispod tačke porudžbine `Pc`.

### 6.1 Ključne oznake SP

| Oznaka | Opis |
|---|---|
| `Q` | Fiksna količina jedne narudžbine |
| `Pc` | Tačka porudžbine — nivo zaliha koji okida narudžbinu |
| `D` | Vreme isporuke (dana) |

### 6.2 Tačka porudžbine

$$Pc = d \cdot D$$

> Kada zalihe padnu na ili ispod `Pc`, automatski se daje narudžbina za `Q` komada.

### 6.3 Stanje zaliha (SP)

$$Z_t = Z_{t-1} + Ulaz_t - Izlaz_t$$

### 6.4 Uslov za narudžbinu

$$\text{Ako } Z_t \leq Pc \Rightarrow \text{naruči } Q$$

### 6.5 Prosečne zalihe (SP)

$$\bar{Z}_{SP} = \frac{\sum Z_t}{T}$$

---

## 7. Troškovi u PP i SP modelima

### 7.1 Troškovi naručivanja

$$TC_s = n \cdot Cs_{fix} + Ulazi_{ukupni} \cdot Cs_{var}$$

> `n` = broj narudžbina tokom perioda; `Ulazi_ukupni` = zbir svih primljenih paketa.

### 7.2 Troškovi držanja

$$TC_h = \bar{Z} \cdot Ch_{var} \cdot T$$

> `Z̄` = prosečne dnevne zalihe; množi se varijabilnim troškom i brojem dana.

### 7.3 Troškovi penala (kazni za nedostatak)

$$TC_p = \text{broj dana deficita} \cdot \bar{\delta} \cdot Cp_{var}$$

> `δ̄` = prosečni dnevni deficit (negativno stanje zaliha); Cp(fix) se dodaje po svakom ciklusu deficita.

### 7.4 Troškovi dolaska / tehničkog pregleda

- **PP model:** `TC_{dolazak} = n_{revizija} · Cena\_po\_dolasku`
- **SP model:** `TC_{tehnički} = T · Tehnički_{dnevni}`

---

## 8. Provere i kontrola

### 8.1 Provera konzistentnosti

$$\theta_1 + \theta_2 = \theta \quad \checkmark$$

$$n \cdot \theta = T \quad \checkmark$$

$$n \cdot Q^* \approx N \quad \checkmark$$

### 8.2 Zbir ulaza i izlaza

$$\sum Ulazi \approx \sum Izlazi \quad \text{(na kraju perioda)}$$

> Razlika je promena stanja zaliha: `Z_{kraj} - Z_{početak}`.

### 8.3 Negativne zalihe (deficit)

Ako je `Z_t < 0` — roba nije isporučena na vreme → aktivira se `Cp`.

U nekim modelima koristi se **karantin** (odložena isporuka) koji uvećava efektivne zalihe u budućim periodima.

---

## 9. Brza referenca — primer vrednosti

| Veličina | Zadatak I | Zadatak II |
|---|---|---|
| T | 60 dana | 30 dana |
| N | 6 000 paketa | 2 400 paketa |
| d = N/T | 100 pak/dan | 80 pak/dan |
| Q* | 346,41 pak | 186,19 pak |
| y | 288,68 pak | — |
| n | 17,32 nar. | 12,89 nar. |
| θ | 3,46 dana | 2,33 dana |
| θ1 | 2,89 dana | — |
| θ2 | 0,58 dana | — |

---

## 10. Redosled izračunavanja (workflow)

```
1. Unesi: T, N, Cs(fix), Cs(var), Ch(fix), Ch(var), Cp(fix), Cp(var)
2. Izračunaj: d = N / T
3. Izračunaj: Q* (EOQ formula sa/bez penala)
4. Izračunaj: y = Q* · Cp(var) / (Ch(var) + Cp(var))
5. Izračunaj: n = N / Q*
6. Izračunaj: θ = T / n
7. Izračunaj: θ1 = y / d
8. Izračunaj: θ2 = θ - θ1
9. Izračunaj troškove: TC_s, TC_h, TC_p
10. Ukupno: TC = TC_s + TC_h + TC_p
11. Simulacija PP: za svaki dan pratiti Z, narudžbine (na RevDan), ulaze (D dana posle)
12. Simulacija SP: za svaki dan proveriti da li Z ≤ Pc → naruči Q
```

---

*Cheatsheet napravljen na osnovu Excel datoteka: `UPRAVLJANJE_ZALIHAMA_osnovni_okvir.xlsx`, `UPRAVLJANJE_ZALIHAMA_I_zadatak.xlsx`, `UPRAVLJANJE_ZALIHAMA_II_zadatak.xlsx`*
