# Python telepítése és emtsv elérése Pythonból

Ebben az útmutatóban a Python telepítéséről lesz szó Windowsra a _Conda_ csomagkezelővel. Amennyiben Pythonunk és Jupyter Labunk (vagy Notebookunk, az egy korábbi verzió), akkor ugorhatunk az emtsvvel foglalkozó fejezethez. Az útmutató alapszintű Python ismereteket előfeltételez.

## Mi micsoda?

### 1. [Python](https://hu.wikipedia.org/wiki/Python_(programoz%C3%A1si_nyelv))

Általános célú programozási nyelv, nagyon népszerű, részben az egyszerű szintaxisa és remek olvashatósága miatt.
Mivel a Python önmagában csak egy programozási nyelv, ezért csomagokat telepítünk, hogy kibővítsük a funkcionalitást.

### 2. [Conda](https://docs.conda.io/en/latest/)

Csomagkezelő. A Conda végzi el a Python telepítést és a csomagok adminisztrálását. Két változatban lehet letölteni: *Ana*conda és *Mini*conda, a különbség csak az előre feltelepített csomagok számában jelentkezik, mi csak néhány csomagot fogunk használni, így Minicondát telepítünk.

### 3. [Jupyter](https://jupyter.org)

Böngészőből elérhető felület Pythonhoz. További előnye, hogy notebookokat hozunk létre kódcellákkal, amin egyszerűen bemutathatjuk a kódunkat.

## Telepítés

1. Miniconda letöltése erről a címről: [https://docs.conda.io/en/latest/miniconda.html](https://docs.conda.io/en/latest/miniconda.html). Válasszuk ki a legmagasabb Python verzióból a 64-bites telepítőt, ez egy .exe fájl, töltsük le.

2. Telepítésnél figyeljünk arra, hogy bepipáljuk az _Add Miniconda3 to my PATH environment variable_ opciót. Minden más maradhat alapértelmezetten a telepítés során.

3. Conda shell megnyitása: Start Menü ➞ Anaconda Prompt (miniconda3)

4. Üssük be a feljövő parancssorba a következő parancsot, mely telepít 3 csomagot és annak a függőségeit. Ne lepődjünk meg, sok csomag kerül telepítésre.

    ``` conda
    conda install -c conda-forge jupyterlab seaborn pandas
    ```

## Jupyter Lab használata

A példákkal illusztrált notebook a _Notebooks/emtsv_usage.ipynb_ helyen van.

1. Indítsuk el a Jupyter Labot az Anaconda Promptból: `jupyter lab`

2. Megnyíilik a Jupyter Launcher. A Jupyter Lab füles elrendezést használ - bal oldalon vannak a Jupyter kezeléséhez tartozó fülek - File browser (_fájlkezelő_), Running kernels (_futó notebookok_), Table of contents (_tartalomjegyzék a notebookhoz_), Extensions (_kiegészítők_).

3. Csináljunk egy új notebookot: kattintsunk a _Notebook_ felirat alatt a _Python 3_ gombra, ilyenkor létrejön egy _Jupyter notebook_.

4. Kaptunk egy üres notebookot egy üres cellával. A cellába már írhatjuk is a Python kódot, amit aztán tudunk is futtatni a ▷ lejátszás gombbal a felső sávon vagy a Shift+Enter billentyűkombinációval. Ilyenkor lefut a kód és létrejön egy új cella. Amennyiben csak a jelen cellát szeretnénk futtatni és nem létrehozni újat, a Ctrl+Enter billentyűkombinációt használjuk.

5. A cellákat lehet rendezni, törölni, klónozni, kettévágni. A Ctrl+S kombinációval vagy a 💾 floppy ikont megnyomva tudunk elmenti, de a Jupyter időnként automatikusan is ment.

6. Amikor végeztünk, elég bezárni a parancssor ablakát, ahonnan indítottunk a Jupyter Labot.

## emtsv elérése Jupyter Labból

1. Indítsuk el a Docker konténert Powershellből: `docker run --rm -p5000:5000 -it mtaril/emtsv`

2. Indítsuk el a Jupyter Labot az Anaconda Promptból: `jupyter lab`

3. A `requests` csomagra szükségünk lesz, így az első cellában importáljuk: `import requests`. Későbbiekben fogunk még egy két-csomagot használni, jó, ha azokat is importáljuk.  Ne felejtsük utána lefuttatni a cellát!

    ```python
    import requests
    import pandas as pd  # for dataframes
    from io import StringIO  # for dataframes
    ```

4. Lehet szöveget vagy fájlt átadni az emtsvnek. A requests csomag segítségével egy kérelmet (requestet) küldünk az emtsvnek, az pedig visszaadja a kimenetet.

    ```python
    text = 'Ki korán kel, aranyat lel.'
    r = requests.post('http://127.0.0.1:5000/tok/morph/pos', data={'text': text})
    ana = pd.read_table(StringIO(r.text))
    ana[['form', 'xpostag']]
    ```

    |    | form    | xpostag            |
    |---:|:--------|:-------------------|
    |  0 | Ki      | [/Prev]            |
    |  1 | korán   | [/Adv]             |
    |  2 | kel     | [/V][Prs.NDef.3Sg] |
    |  3 | ,       | [Punct]            |
    |  4 | aranyat | [/N][Acc]          |
    |  5 | lel     | [/V][Prs.NDef.3Sg] |
    |  6 | .       | [Punct]            |

5. Fájl betöltése és feldolgozása:

    ```python
    infile_path = 'C:/Users/Daniel/Documents/Ablonczy_utf8.txt'
    outfile_path = 'ablonczy_pos.tsv'
    # always specify UTF8 if input file is UTF8!
    with open(infile_path, 'r', encoding='UTF8') as infile, open(outfile_path, 'w', encoding='UTF8') as outfile:
        r = requests.post('http://127.0.0.1:5000/tok/morph/pos', files={'file': infile})  # we hand the file to the emtsv server
        outfile.write(r.text)
    ```

## Tippek, trükkök, példákkal illusztrálva a notebookban

- Amíg kicsik a fájlok (a bemeneti fájl néhány megabájtos), addig a `pandas` használata nagyon meggyorsítja az adatelemzést; nagyobb fájloknál magunknak kell gondoskodni a kimenet feldolgozásáról.

- Több fájl összefésülésénél figyelni kell arra, hogy az emtsv requestenként ad vissza fejlécet - azaz 10 fájl (vagy 10 egyesével elküldött mondat) esetén 10 fejléc lesz - az első új sorig (`'\n'`) való szűréssel meg tudjuk oldani ezt a problémát.

- Ha a szövegünkben vannak whitespace karakterek, akkor beolvasásnál ezekkel meg fog gyűlni a bajunk - tab szerint szeretnénk szeparálni, de néhány sor értelemszerűen hosszabb lesz, hiszen maga a szó is egy tab - tehát beolvasásnál figyeljünk arra, hogy megfelelően széles-e a sor, és ha nem, akkor döntsünk, hogy eldobjuk-e, vagy utófeldolgozzuk.
