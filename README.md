# Airbnb būstų nakties kainų Niujorke analizė

## Projekto aprašymas

Šiame projekte analizuojama, kaip Airbnb būstų nakties kainos Niujorke priklauso nuo būsto charakteristikų ir rajono, kuriame jis yra.

Analizei taikomas **hierarchinis tiesinis modelis (HLM)**, leidžiantis atsižvelgti į hierarchinę duomenų struktūrą: individualūs Airbnb skelbimai (I lygmuo) yra suskirstyti į rajonus (II lygmuo).

## Tyrimo tikslas ir uždaviniai

Pagrindinis tikslas buvo nustatyti, kaip Airbnb būsto nakties kaina Niujorke priklauso nuo paties būsto savybių (I lygmuo) ir nuo rajono, kuriame jis yra (II
lygmuo).

Tyrimo uždaviniai:
1. Pirminė duomenų analizė.
2. Besąlyginio atsitiktinio postūmio modelio sudarymas ir ICC apskaičiavimas.
4. Hierarchinių modelių sudarymas bei modelių palyginimas pagal AIC ir liekanų dispersiją.
5. Galutinio modelio parinkimas bei jo liekanų normalumo vertinimaa.

## Duomenys

Naudojamas viešai prieinamas duomenų rinkinys [New York City Airbnb Open Data](https://www.kaggle.com/datasets/dgomonov/new-york-city-airbnb-open-data), paskelbtas platformoje Kaggle.

Duomenų rinkinyje pateikiama informacija apie 2019 m. Niujorke veikusius Airbnb būstų skelbimus.

| Aprašymas | Reikšmė |
|---|---:|
| Pradinis stebėjimų skaičius | 48 895 |
| Galutinis stebėjimų skaičius | 48 884 |
| Kintamųjų skaičius | 16 |
| Rajonų skaičius | 221 |

## Hierarchinė duomenų struktūra ir kintamieji

### I lygmuo – būsto kintamieji

| Kintamasis | Aprašymas |
|---|---|
| `price` | Nakvynės kaina USD; modeliuojama logaritmuota kaina |
| `room_type` | Būsto tipas: visas būstas, privatus arba bendras kambarys |
| `availability_365` | Dienų per metus, kuriomis būstas yra prieinamas, skaičius |
| `number_of_reviews` | Atsiliepimų skaičius |
| `minimum_nights` | Minimalus nakvynių skaičius |
| `calculated_host_listings_count` | Šeimininko skelbimų skaičius |

### II lygmuo – rajonų kintamieji

| Kintamasis | Aprašymas |
|---|---|
| `dist_km` | Rajono centro atstumas iki Times Square kilometrais |
| `avg_reviews` | Vidutinis atsiliepimų skaičius rajone |

Antrojo lygmens kintamieji sukurti agreguojant pirmojo lygmens duomenis.

## Duomenų paruošimas

Atliekant duomenų paruošimą:

1. Pašalinti 11 būstų, kurių kaina buvo lygi 0 USD.
2. Kainos kintamajam pritaikyta natūraliojo logaritmo transformacija (`log_price`).
3. Apskaičiuotas vidutinis atsiliepimų skaičius kiekviename rajone.
4. Apskaičiuotas rajonų centrų atstumas iki Times Square, naudojant Haversine formulę.
5. Rajonų lygmens kintamieji prijungti prie būstų duomenų.

Logaritminė kainos transformacija pasirinkta dėl stiprios dešiniosios kainų pasiskirstymo asimetrijos.

## Besąlyginis modelis ir ICC

Besąlyginis modelis sudarytas siekiant įvertinti, kokią kainų dispersijos dalį paaiškina skirtumai tarp rajonų.

Gautas intraklasinės koreliacijos koeficiento (ICC) įvertis:

**ICC ≈ 0,292**

Tai reiškia, kad maždaug 29 % logaritmuotos būsto kainos dispersijos lemia skirtumai tarp rajonų. Šis rezultatas pagrindžia hierarchinio tiesinio modeliavimo taikymą.

## Galutinis HLM modelis

Galutiniame modelyje naudojami šie fiksuotieji efektai:

- Būsto tipas (`room_type`)
- Prieinamumas (`availability_365`)
- Atsiliepimų skaičius (`number_of_reviews`)
- Minimali nakvynių trukmė (`minimum_nights`)
- Atstumas iki Times Square (`dist_km`)
- Vidutinis atsiliepimų skaičius rajone (`avg_reviews`)

Atsitiktiniai efektai įtraukti rajono postūmiui ir būsto tipo koeficientams.

Šeimininko skelbimų skaičiaus kintamasis pašalintas, nes jo praktinė įtaka kainai buvo labai maža.

### Galutinio modelio įverčiai

| Kintamasis | Koeficientas |
|---|---:|
| Postūmis | 5,2964 |
| Privatus kambarys | -0,7070 |
| Bendras kambarys | -1,0364 |
| `availability_365` | 0,000771 |
| `number_of_reviews` | -0,000741 |
| `minimum_nights` | -0,001986 |
| `dist_km` | -0,023957 |
| `avg_reviews` | -0,005994 |

Visi galutinio modelio fiksuotieji efektai statistiškai reikšmingi (p < 0,001).

## Rezultatų interpretacija

1. Privatus kambarys yra maždaug **51 % pigesnis** už visą būstą.
2. Bendras kambarys yra maždaug **65 % pigesnis** už visą būstą.
3. Padidėjus prieinamumui 100 dienų, kaina padidėja maždaug **8 %**.
4. Padidėjus atsiliepimų skaičiui 100, kaina sumažėja maždaug **7 %**.
5. Padidėjus minimaliai nuomos trukmei nuo 1 iki 30 nakvynių, kaina sumažėja maždaug **6 %**.
6. Kiekvienas papildomas kilometras nuo Times Square siejamas su maždaug **2,4 % mažesne kaina**.
7. Rajone vidutiniam atsiliepimų skaičiui padidėjus 10, kaina sumažėja maždaug **6 %**.
8. Būsto tipo ir kainos ryšys taip pat skiriasi priklausomai nuo rajono.

## Modelių palyginimas

| Modelis | AIC | I lygmens dispersijos sumažėjimas |
|---|---:|---:|
| Besąlyginis | 88 614,5 | – |
| Modelis 1 | 66 178,1 | 36,7 % |
| Modelis 2 | 65 568,3 | 37,9 % |
| Galutinis modelis (Modelis 3) | 65 590,4 | 37,8 % |

Lyginant galutinį modelį su besąlyginiu:

- AIC sumažėjo maždaug **23 024 punktais**.
- I lygmens liekanų dispersija sumažėjo maždaug **38 %**.

Galutiniame modelyje atsisakyta šeimininko skelbimų skaičiaus kintamojo dėl labai mažos jo praktinės įtakos.

## Liekanų analizė

Liekanų normalumas vertintas naudojant histogramą ir Q-Q grafiką.

Analizė parodė, kad daugumos stebėjimų liekanos apytiksliai atitinka normalųjį skirstinį. Vis dėlto dešiniojoje uodegoje yra išskirčių, susijusių su neįprastai brangiais būstais. Dėl šių stebėjimų statistinio reikšmingumo rezultatus reikėtų vertinti atsargiau.

## Išvados

- Atlikta analizė parodė, kad Niujorko Airbnb būstų kainoms svarbios tiek individualios būsto savybės, tiek rajono charakteristikos.
- Apie 29 % kainų dispersijos lemia rajonas.
- Vienas svarbiausių kainos veiksnių yra būsto tipas.
- Būsto prieinamumas teigiamai susijęs su kaina.
- Atsiliepimų skaičius ir minimali nuomos trukmė neigiamai susiję su kaina.
- Kuo toliau būstas yra nuo Times Square, tuo mažesnė jo kaina.
- Galutinis hierarchinis modelis gerokai geresnis už besąlyginį modelį.

## Duomenų šaltinis

Dgomonov. (2019). *New York City Airbnb Open Data*. Kaggle.

https://www.kaggle.com/datasets/dgomonov/new-york-city-airbnb-open-data

---

**Autoriai:** Naglis Bražėnas, Justina Seiliūnaitė
