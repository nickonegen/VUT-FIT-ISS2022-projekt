# ISS Projekt 2022/23

*Honza Brukner a Honza Černocký, ÚPGM FIT VUT*

## Studijní etapa

*kopírováno z [veřejných stránek](https://www.fit.vutbr.cz/study/courses/ISS/public/#proj)*

Tahle etapa je zaměřena na praktické procvičení signálů. Doporučujeme všechna cvičení projet a snažit se pochopit vše o zpracování signálu:

1. [Python notebooky](https://www.fit.vutbr.cz/study/courses/ISS/public/proj_studijni_etapa/), které jsou výsledkem bakalářské práce ##### ##### v roce ####/##. Doporučuji si pečlivě přečíst instrukce v souboru README.md a update-ovat Python prostředí.
2. [Stará verze studijní etapy](https://www.fit.vutbr.cz/study/courses/ISS/public/proj_studijni_etapa_old_Matlab/), pokud si chcete procvičit základy MATLABu.
3. Některé části [Python notebooků ##### ###########](https://www.fit.vutbr.cz/~izmolikova/ISS/project/)

## Úvod projektu

V projektu bude vaším úkolem vytvořit co nejvěrnější syntetické piano, jehož zvuková charakteristika bude komprimovaná pouze z několika paramatrů. K dispozici máte nahrávky jednotlivých tónů, které bude potřeba analyzovat a následně vzgenerujete skladbu na základě dodaného souboru.

Projekt je individuální a je ho možno řešit v Python-u, MATLAB-u, Octave, C nebo libovolném programovacím nebo skriptovacím jazyce. Je možné použít jakékoliv knihovny. **Důležitý je výsledek**, nemusíte se trápit o komentaci kódu a ošetřování chybových stavů - ale **kód musí prokazatelně produkovat výsledky obsažené ve Vašem protokolu**.

## Vstup

V souboru [klavir.wav](https://www.fit.vutbr.cz/study/courses/ISS/public/proj2022-23/klavir.wav) máte k dispozici WAV soubor se všemi tóny klavíru. Vzorkovací frekcence je 48 kHz a nahrávka má jeden kanál. Tóny jsou od C2 (MIDI 24) do C9 (MIDI 108) - celkem 85, každý má dvě sekundy. Přehled MIDI značení a frekvencí tónů najdete v souboru [midi.txt](https://www.fit.vutbr.cz/study/courses/ISS/public/proj2022-23/midi.txt). Dále, v souboru [xlogin00.txt](https://www.fit.vutbr.cz/study/courses/ISS/public/proj2022-23/personal/xlogin00.txt) (použijte Váš login) máte označenú tónů, na kterých budete demonstrovat průběžné řešenú projektu.

## Zadání

Úspěšné řešení těcho bodů zadání vede k plnému počtu bodú - tedy 18.

Všechny grafi musí mít popsané osy. Pokud vykreslujete v časové doméně, osa *x* bude zobrazovat čas v (mili-/mikro-/...)sekundách; pokud ve spektrální, osa *x* bude ve frekvencích Hz.

Pokud nebude uvedeno jinak, spektrum zobrazujte jako log PSD (log. druhé mocniny abs. hodnot DFT) od 0 Hz do poloviny vzorkovací frekvence. Jelikož signál ustředňujeme, a nultý koeficient DFT bude 0 (log 0 = -inf), můžete pro potřeby vykreslování přidat k log PSD nějakou malou konstantu (třeba 10e-5).

### Základy (2 body)

Načtete všechny signály a vyberte 0.5 sek. dlouhý úsek ze stabilní části signálu, tedy té, kde zní pouze tón a už není slyšet úder kladívka (doporučení: přeskočit první 0.25s signálu). Pokud budete pracovat v Pythonu, můžete načtení provest následovně:

<details>
<summary>Zobrazit kód</summary>

```python
import numpy as np
import soundfile as sf
MIDIFROM = 24
MIDITO = 108
SKIP_SEC = 0.25
HOWMUCH_SEC = 0.5
WHOLETIME_SEC = 2.0
howmanytones = MIDITO - MIDIFROM + 1
tones = np.arange(MIDIFROM, MIDITO + 1)
s, Fs = sf.read('klavir.wav')
N = int(Fs * HOWMUCH_SEC)
Nwholetone = int(Fs * WHOLETIME_SEC)
xall = np.zeros((howmanytones, N))
samplefrom = int(SKIP_SEC * Fs)
sampleto = samplefrom + N
for tone in tones:
    x = s[samplefrom:sampleto]
    x = x - np.mean(x)
    xall[tone,:] = x
    samplefrom += Nwholetone
    sampleto += Nwholetone
```

</details>

Zobrazte tři periody vašich tří tónů v ustálené části a spočítejte (a vykreslete) DFT celého 0.5 sek. dlouhého úseku. Vaše tóny uložte jako `audio/a_orig.wav`, `audio/b_orig.wav` a `audio/c_orig.wav`.

### Určení základní frekvence (3 body)

Spočítejte základní frekvenci všech tónů a srovnejte je s frekvencemi definovanými MIDI. Základní frekvenci můžete spočítat pomocí autokorelace nebo DFT.

Pro vaše tři tóny zobrazte graf s použitou metodou a vyznačte v něm kde a jak jste nalezli *f0*, pokud jste použili nějaký výpočet (uveďte). Pokud vidíte rozdíly mezi očekávanou (MIDI) a skutečnou frekvencí, komentujte z čeho můžou plynout (rozladění piana, přesnost metody (jaká je?),...).

Hint: Autokorelace funguje lépe pro tóny s nižší frekvencí, kde má DFT tendenci selhávat a naopak. Máte k dispozici MIDI frekvenci, pomocí kreté si můžete skontrolvoat, jestli jste frekvenci správně určili.

### Zpřesnění odhadu základní frekvence f_0 (3 body)

Pomocí DTFT (ne DFT!) zpřesněte odhad skutečné základní frekvence všech tónů.

Můžete například vypočítat koeficienty pro frekcence rozložené 100 [centů](https://cs.wikipedia.org/wiki/Cent_(hudba)) okolo nejblyžší MIDI frekcence. Další možnost je použít frekvence v rozpětí 2 koeficientů DFT kolem odhadovaného základního tónu. U některých tónů (hlavně na nižších frekvencích) se stává, že je první koeficient FŘ nižší, než ten druhý - můžete tedy také vyhledat, na které frekvenci leží druhý koeficient a tu potom podělit 2. Pokud vám příliš překáží laloky sinc, způsobené ostrým obdélníkovým oknem, můžete použít okénkovou funkci, např. `np.hamming`.

Srovnejte takto získaný odhad s původním, napište, kterou možnost používáte, případně jestli je kombinujete a uveďte, jakým způsobem DTFT implementujete.

### Reprezentace klavíru (3 body)

Reprezentujte každý tón použitím 10 floating point čísel.

Možný postup: Aproximujte Fourierovu řadu (FŘ) pomocí DTFT, spočítejte koeficienty na násobcích základní frekvence (1f_0 ... 5f_0), uložte modul a fázi, alternativně můžete spočítat pouze moduly pro prvních 10 násobků a neukládat fáze. Jelikož pracujeme s reálnými signály, opět vyhledejte přesnou polohu koeficientu z intervalu okolo očekávané frekvence.

Zobrazte modul DFT vašich tří tónů, do stejných grafů vyznačte koeficienty (budou to body na křivce), frekvenční osu vykreslete od 0 do 11 × nejbližší MIDI frekvence v Hz (v protokolu budou 3 grafy, jeden pro každý tón).

### Syntéza tónů (3 body)

Syntetizujte signály odpovídající Vašim třem tónům. Použijte stejnou vzorkovací frekvenci jako má původní nahrávka.

Pro syntézu můžete využít FŘ (uvědomte si, že `c−i = c∗i`; výsledek je reálný, ale díky nepřesnostem výpočtu bude pořád komplexní číslo, imaginární části se můžete zbavit pomocí np.real()) nebo využijte prosté sčítání cosinusovek (jak vyřešíte amplitudu?).

Uložte 1 sekundu vygenerovaného signálu pro každý z vašich tří tónů do souborů `audio/a.wav`, `audio/b.wav`, a `audio/c.wav`.

Srovnejte v grafu 10 period vámi vygenerovaného a originálního signálu na vašich třech tónech. Signály synchronizujte (ručně, korelací, ... ) tak, aby začínaly se stejnou fází. Komentujte rozdíly.

### Generování hudby (3 body)

V [skladba.txt](https://www.fit.vutbr.cz/study/courses/ISS/public/proj2022-23/skladba.txt) máte k dispozici skladbu zpracovanou z MIDI souboru s řádky v následujícím formátu, kde každý řádek odpovídá jednomu stisknutí klávesy:

```text
od [ms]   do [ms]   MIDI    hlasitost
```

Vygenerujte skladbu. Pozor, nahrávka je polyfonní, zní v ní tedy několik tónů současně. Vytvořte dvě nahrávky, jednu se vzorkovací frekvencí 8000 Hz a druhou s 48000 Hz. Z obou uložte prvních 10 s jako `audio/out_8k.wav` a `audio/out_48k.wav`. Pro generování na 8000 Hz dejte pozor, abyste neporušili vzorkovací teorém - nesmíte generovat harmonické nad F_s/2.

Pokud si budete chtít vygenerovat nějaké skladby pro potěšení, můžete se podívat zde: [https://github.com/Lemlak/ISS22_table_generator](https://github.com/Lemlak/ISS22_table_generator)

### Spektrogram (1 bod)

Zobrazte spektrogram prvních 10 s vygenerovaného signálu pro obě vzorkovací frekvence, komentujte, co vidíte. Použijte velikost okna 0.03 ms, překryv 0 ms a počet vzorků DFT 2048 pro 44.1 kHz a 512 pro 8 kHz. Můžete využít knihovní funkce scipy.signal.stft nebo scipy.signal.spectrogram.

## Odevzdání

Projekt se bude odevzdávat do [IS VUT](https://www.vut.cz/studis/student.phtml), do konce semestru - tedy do neděle 18.12.2022 23:59. Odevdávají se dva soubori:

- `xlogin00.pdf` - protokol řešení
  - V záhlaví uveďte své jméno, příjmení a login.
  - Pak zodpovězte otázky v zadání pomocí obrázků, komentářů a numerických hodnot.
  - U každé otázky uveďte stručný postup - může se jednat o kousek okomentovaneho kódu, komentovanou rovnici nebo jen text - není nutné kopírovat celý kód do protokolu, jen relevantní část. Teorií či zadání není nutné opisovat, soustřeďte se přímo na řešení.
  - Pokud využijete zdroje mimo standardních materiálů (co jsou přednážky, cvičení a studijní etapa projektu ISS), uveďte, odkud jste čerpali (např. článek, skripta, dokumentace numpy, MATLAB, scipy,...)
  - Protokol je možné psát v libovolném systému (MS-Word, Libre Office, Latex,...), nebo i čitelně rukou. Pokud budete protokol tvořiť přímo jako Jupyter notebook (doporučeno), prosíme Vás o jeho export do PDF.
  - Protokol může být v češtině, slovenštině nebo v angličtině. Budete-li psát anglicky, výsledný soubor nazvěte `xlogin00_e.pdf`.
- `xlogin00.tar.gz` - komprimovaný archiv řešení s následujícími adresářmi:
  - `/src` (zdrojový kód) - může se jednat o jeden soubor (`reseni.py` nebo `reseni.ipynb`), o více souborů/skriptů nebo o adresářovou strukturu
  - `/audio` (zvukové soubory) - ve formátu WAV, bit.šířka 16 bitů, bez komprese se zadanou vzorkovací frekvencií.

Projekt je **samostatná práce**. Vaše zdrojové kódy budou křížově korelovány, a v případě silné podobnosti budou vyvozeny příslušné závěry. Silná korelace s kódy ze studijní etapy projektu ISS nebo Python notebooků studijní opory či přednášek je v pořádku.
