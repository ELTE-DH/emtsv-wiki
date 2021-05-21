# emtsv használata Powershellben

Windowson kicsit körülményes a Powershellből való futtatás, de meg lehet oldani.
A bemeneti fájlnak UTF-8-as kódolásúnak kell lennie, különben nem fog működni az alábbi leírás.

1. Nyissunk meg egy Powershell ablakot

2. Navigáljunk arra a helyre, ahol a fájlaink vannak.

3. Állítsuk be a Powershell szövegkódolását: másoljuk be az alábbi négy sort, majd nyomjunk Entert.

```powershell
$InputEncoding = [System.Text.UTF8Encoding]::new()
$OutputEncoding = [System.Text.UTF8Encoding]::new()
[console]::InputEncoding = [System.Text.UTF8Encoding]::new()
[console]::OutputEncoding = [System.Text.UTF8Encoding]::new()
$Utf8NoBomEncoding = New-Object System.Text.UTF8Encoding $False
```

4. Ezután a bemeneti fájlt már át tudjuk adni a Docker imagenek az alábbi paranccsal, a kimenetet pedig elmentjük egy változóba.

```powershell
$MyRawString = cat -Encoding utf8 .\bemeneti_fajl_utf8.txt | docker run -i mtaril/emtsv tok,morph,pos
```

5. Ezután már csak ki kell írni a kimenetet. Az első sorral frissítjük a második sorban lévő parancs elérési útvonalát, a második sorral pedig ténylegesen kiírjuk a tartalmat.

```powershell
[System.Environment]::CurrentDirectory = (Get-Location).Path
[System.IO.File]::WriteAllLines('kimeneti_fajl_utf8.tsv', $MyRawString, $Utf8NoBomEncoding)
```

Ezzel készen is vagyunk, Powershellben futtattuk egy fájlra az emtsv-t.