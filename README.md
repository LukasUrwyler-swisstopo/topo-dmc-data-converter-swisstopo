# DMC Data Converter

> [!NOTE]
> Dieses Repository ist ein Wegweiser. Der Quellcode wird in der swisstopo-Organisation weiterentwickelt:
>
> **https://github.com/swisstopo/topo-dmc-data-converter**
>
> Das Repository ist privat. Zugriff haben Mitglieder der swisstopo GitHub-Organisation.

## Worum es geht

Desktop-Tool mit grafischer Oberfläche für die Aufbereitung von Luftbilddaten der Kamera **Leica DMC-4** aus **Reality Studio** (Esri). True-Orthophotos und photogrammetrische Punktwolken liegen dort als technische 200-m-Kacheln vor. Das Tool überführt sie in lieferfertige 1-km-Kacheln im Schweizer Landeskoordinatensystem, bereit für die Publikation im Geodatenwarehouse (GDWH) und via STAC.

Entwickelt bei swisstopo für die Produktion von Spezialbefliegungen.

## Funktionen

| Tab | Was er macht |
|---|---|
| **DOP - TIFFconverter** | 200-m-Orthophoto-Kacheln mosaikieren, auf die gültige Fläche clippen und ins 1-km-Grid zuschneiden. Bandauswahl RGBN, RGB, NRG (Falschfarben-Infrarot) oder NRGB. Optional ein QC-Mosaik als COG. |
| **DSM - LASconverter [LHN95]** | Punktwolken-Kacheln je 1-km-Zelle mergen, auf AOI und Zelle croppen und optional ausdünnen. Ausgabe als LAS 1.4 / PF7 mit RGB, vorbereitet für die Höhentransformation mit GeoSuite/REFRAME. |
| **DSM - LASconverter [LN02]** | Die nach LN02 transformierten Kacheln requantisieren, CRS-Tags setzen und vollständig validieren. Optional DSM, Hillshade und ein QC-COPC der ganzen AOI. |
| **Create DSM-Raster** | Aus beliebigen LAS/LAZ-Kacheln ein DSM und einen Hillshade rechnen (zellweise IDW, Lückenfüllung, AOI-Maske). |
| **Create COGTIFF** | Beliebige TIFF-Kacheln zu einem Cloud Optimized GeoTIFF zusammenführen. |
| **Create COPC** | Beliebige LAS/LAZ-Kacheln zu einer Cloud Optimized Point Cloud zusammenführen, optional ausgedünnt. |

## Ablauf bei den Punktwolken

```
technische 200-m-LAZ  (Reality Studio)
      │
      ▼   Tab [LHN95]         AOI-Crop · Thinning · 1-km-Grid
1-km-Kacheln (LHN95)
      │
      ▼   GeoSuite / REFRAME  Höhentransformation LHN95 -> LN02 (ausserhalb des Tools)
1-km-Kacheln (LN02)
      │
      ▼   Tab [LN02]          Header · CRS-Tags · Validierung · optional DSM, Hillshade, COPC
Lieferung GDWH / STAC
```

Die Höhentransformation macht bewusst ausschliesslich die amtliche Software GeoSuite/REFRAME. Im Tool werden Z-Werte nirgends umgerechnet.

## Technik

- **Python** mit **tkinter**-GUI. Die Verarbeitung läuft getrennt davon in der OSGeo4W- bzw. QGIS-Python-Umgebung.
- **GDAL/OGR** für Raster und Vektoren, **PDAL** für Punktwolken, **untwine** für COPC.
- Parallele Verarbeitung über mehrere Prozesse, zellweise gerechnet, damit auch grosse Projekte im Arbeitsspeicher Platz haben.
- Formate: GeoTIFF, COG, LAS 1.4 / PF7, LAZ, COPC.
- Raumbezug: **EPSG:2056** (CH1903+ / LV95), Höhen **LHN95** (EPSG:5729) bzw. **LN02** (EPSG:5728).

## Qualitätssicherung

Ein paar Punkte, auf die das Tool besonders achtet:

- **CRS-Tags byte-exakt** nach Referenzkachel, weil PDAL und LAStools in den getesteten Versionen abweichende oder unvollständige Tags schreiben.
- **NoData und Nahes Infrarot:** Wasser hat im NIR echte Nullwerte. Das Tool unterscheidet sie von echtem NoData, damit Seen nicht als Löcher in der Lieferung landen.
- **Alpha-Korrektur:** Ein fälschlich als Alpha getaggtes NIR-Band würde RGB-Werte löschen. Das wird vor der Verarbeitung korrigiert.
- **Atomares Schreiben und Validierung** jeder Kachel (Punktzahl, Ausdehnung, Header, Klassifizierung, Raumbezug), bevor sie an ihren Platz kommt.
- **Platz-Check** vor dem COPC-Bau, damit ein Lauf nicht erst nach Stunden am vollen Laufwerk scheitert.

## Autor

Lukas Urwyler, swisstopo (Bundesamt für Landestopografie)
