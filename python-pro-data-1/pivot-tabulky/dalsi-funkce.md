## Další funkce pro tvorbu pivot tabulek

Pivot tabulky velmi často provádíme v kombinaci s nějakou agregací. Uvažujme například, že by nás zajímal průměrný obsah výživných látek za jednotlivé kategorie, nikoli za konkrétní potraviny. 

```py
import pandas as pd

food_nutrient = pd.read_csv("food_nutrient.csv")
branded_food = pd.read_csv("branded_food.csv")
food = pd.concat([pd.read_csv("food_sample_100.csv"), pd.read_csv("food_other.csv")], ignore_index=True)
food_merged = pd.merge(food, food_nutrient, on="fdc_id")
food_merged = pd.merge(food_merged, branded_food, on="fdc_id")
```

Protože v datech máme obrovské množství různých kategorií i výživných látek, u dat provedeme filtrování. Výběr kategorií s největším počtem potravin jsme již provedli v lekci o vizualizacích. Nyní provedeme výběr výživných látek. Pro naši kontingenční tabulku si vybereme tyto výživné látky:

- `Protein` (bílkoviny),
- `Sodium, Na` (sodík, Na),
- `Total lipid (fat)` (lipidy (tuky)),
- `Carbohydrate, by difference` (sacharidy, měřené rozdílově),
- `Sugars, total including NLEA` (cukry, celkem včetně NLEA),
- `Fatty acids, total saturated` (nasycené mastné kyseliny, celkem),
- `Cholesterol` (cholesterol),
- `Fiber, total dietary` (vláknina, celková stravová),
- `Calcium, Ca` (vápník, Ca),
- `Iron, Fe` (železo, Fe).

Nejprve si vyzkoušíme funkci `pivot_table()`. Pozor, jedná se o odlišnou funkci, než kterou jsme využívali v předchozí lekci. Funkci `pivot_table` určíme pět parametrů, tj. o jeden parametr víc, než který jsme zadávali funkci `pivot()`.

- Prvním parametrem (`data`) určujeme tabulku, se kterou bude funkce pracovat.
- Druhý parametr (`value`) obsahuje název sloupce, ze kterého budou čteny hodnoty, které budou "ve vnitřní části" tabulky. V tomto případě již může být pro každou kombinaci název řádku a názvu sloupce více hodnot, protože funkce počítá s provedením agregace.
- Třetí parametr (`index`) slouží jako popisek řádků. V našem případě zvolíme sloupeček `"branded_food_category"`, tj. název kategorie.
- Čtvrtý parametr (`columns`) bude použit k vytvoření nových sloupečků. Sem doplníme sloupec `name`.
- Jako pátý parametr (`aggfunc`) je třeba vložit funkce, která bude použita k agregaci hodnot. Protože předpokládáme, že pro každou kombinaci kategorie a výživné látky budeme znát více hodnot (protože v každé kategorii máme spousty potravin), je třeba použít nějakou funkci, která z těchto hodnot vypočte jedno číslo. Jedná se obdobu agregace, na což odkazuje i název parametru. Pokud bychom chtěli spočítat průměrnou hodnotu, můžeme například použít funkci `mean()` z modulu `numpy`. Pozor na to, že píšeme pouze název funkce **bez závorek**. Neprovádíme totiž volání funkce, to dělá funkce `pivot_table` za nás.

Na rozdíl od funkce `pivot()` nemusíme parametry psát jako pojmenované. 

```py
import numpy as np

pd.pivot_table(food_merged, "amount", "branded_food_category", "name", np.mean)
```

Z výsledné tabulky vidíme, jak se liší průměrné množství výživných látek v jednotlivých potravinách. Kupříkladu sýry jsou poměrně bohaté na vápaník a cholesterol, naopak je v nich poměrně málo vlákniny.

Alternativou k funkci `pivot_table()` je funkce `crosstab()`. Ta se liší od funkce `pivot_table()` především v tom, že jí zadáváme data jako série hodnot, nikoli jako tabulku a následně názvy sloupců.

```py
pd.crosstab(food_merged["branded_food_category"], food_merged["name"], food_merged["amount"], aggfunc=np.mean)
```

### Standardizace a teplotní mapa

Kontingenční tabulka je časově náročná na čtení, především v případě, že má poměrně hodně řádků nebo sloupců. Pro rychlý přehled může být užitečnější typ vizualizace označovaný jako :term{cs="teplotní mapa" en="heat map"}. Ten převede hodnotu na barevnou škálu. V teplotní mapě můžeme rychle nalézt především výrazně nadprůměrné či podprůměrné hodnoty. U našich dat ale může být problém v tom, že máme v různých sloupcích řádově odlišné hodnoty. Je to samozřejmě ovlivněno tím, že některé výživné látky jsou zobrazné v odlišných jednotkách (množství látky v gramech a miligramech na 100 gramů potraviny). Problém bychom nevyřešili ani převodem na stejné jednotky. Vápníku nebo sodíku totiž bude v potravinách většinou řádově méně než proteinů nebo cukrů.

Problém ale můžeme vyřešit pomocí procesu označovaného jako {cs="normalizace" en="normalization"}. Normalizace dat v kontextu statistiky a zpracování dat znamená převedení různých rozsahů hodnot na společnou škálu, čímž se usnadňuje jejich srovnání a analýza. Tento proces pomáhá odstranit zkreslení v datech způsobená různými měřítky a umožňuje lépe identifikovat vzory a vztahy mezi proměnnými. Normalizace je klíčová pro efektivní algoritmické zpracování, například {cs="strojovém učení" en="machine learning"} a datové analýze.

Prakticky normalizace obsahuje dva kroky:

- Od hodnot odečteme průměr. Tím pádem se normalizované hodnoty budou pohybovat kolem nuly. Pokud bude nějaká normalizovaná hodnota záporná, v původních datech byla podprůměrná. Pokud bude normalizovaná hodnota kladná, v původních datech byla nadprůměrná.
- Vydělíme data jejich variabilitou, konkrétně směrodatnou odchylkou. Tím data převedeme na stejné jednotky. Data se budou pohybovat ve stejných jednotkách, ať už byla původní data v desetinných čísel nebo v minionech.

Nejprve tedy provedeme normalizaci dat.

```py
food_pivot_norm = (food_pivot - food_pivot.mean()) / food_pivot.std()
```

A ve druhém kroku vytvoříme teplotní mapu.

```py
import seaborn as sns
ax = sns.heatmap(food_pivot_norm)
ax.set(xlabel="Výživná látka", ylabel="Kategorie", title="Množství průměrných látek dle kategorií")
```

Pro lepší čitelnost můžeme změnit výchozí barevnou škálu pomocí parametru `cmap`. Můžeme použít například škálu `"Blues"`, která je postavená na sytosti modré barvy.
