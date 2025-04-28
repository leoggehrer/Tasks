# POSE 7ABIF/7ACIF

---

## PayWithFun

**Lehrziele:**  

- Umsetzung der SE-Architektur mit dem [SETemplate](https://github.com/leoggehrer/SETemplate)
- Prüfen von Geschäftsregeln
- Importieren von csv-Daten

## Einleitung

Das Projekt ***SEPayWithFun*** ist eine datenzentrierte Anwendung zur Verwaltung von Kreditkarten-Zahlungen. Diese Anwendung soll mit dem **SE-Template** erstellt werden.

Zu entwickeln ist das **Backend-System** mit der Datenbank-Anbindung und eine *Avalonia MVVM-Anwendung* zur Verwaltung der Zahlungsdaten.

## Datenmodell und Datenbank  

Das Datenmodell für ***PayWithFun*** hat folgenden Aufbau:

```txt

+-------+--------+ 
|                | 
|    Payment     + 
|                | 
+-------+--------+ 

```

| Name           | Type     | MaxLength | Nullable | Unique | Db-Field | Access |  
|----------------|----------|-----------|----------|--------|----------|--------|  
| IdType         | GUID     | ---       | ---      | ---    | Yes      | R      |  
| CardNumber*    | String   | 16        | No       | No     | Yes      | RW     |  
| ExecutionTime  | DateTime | ---       | No       | No     | Yes      | RW     |  
| TurnoverTime*  | DateTime | ---       | No       | No     | Yes      | RW     |  
| DealerName     | String   | 128       | No       | No     | Yes      | RW     |  
| DealerLocation | String   | 128       | No       | No     | Yes      | RW     |  
| Turnover       | decimal  | ---       | No       | No     | Yes      | RW     |  
| Note           | String   | 1024      | Yes      | No     | Yes      | RW     |  

`ExecutionTime`..............Durchführungsdatum

`TurnoverTime`...............Umsatzdatum

*............................Muss eindeutig sein

## Aufgaben  

### Geschäftsregeln

Das System muss einige Geschäftsregeln umsetzen. Diese Regeln werden im Backend implementiert und müssen mit UnitTests überprüft werden.

> **HINWEIS:** Unter **Geschäftsregeln** (Business-Rules) versteht man **elementare technische Sachverhalte** im Zusammenhang mit Computerprogrammen. Mit *WENN* *DANN* Scenarien werden die einzelnen Regeln beschrieben.  

Für den ***SEPayWithFun*** sind folgende Regeln definiert:

| Rule | Subject | Type   | Operation | Description | Note |
|------|---------|--------|-----------|-------------|------|
|**A1**| Payment |        |           |             |      |
|      |         |**WENN**|           | eine Zahlung erstellt oder bearbeitet wird, | |
|      |         |**DANN**|           | muss die `CardNumber` festgelegt sein und gültig sein (die Regeln finden Sie im Abschnitt Prüfung der Kreditkarten-Nummer). | |
|**A2**| Payment |        |           |             |      |
|      |         |**WENN**|           | eine Zahlung erstellt oder bearbeitet wird, | |
|      |         |**DANN**|           | darf `TurnoverTime` nicht jünger als `ExecutionTime` sein. | |
|**A3**| Payment |        |           | | |
|      |         |**WENN**|           | eine Zahlung erstellt oder bearbeitet wird, |  |
|      |         |**DANN**|           | dann muss `CardNumber`und `TurnoverTime` eindeutig sein. | |
|**B1**| Payment |        |           | | |
|      |         |**WENN**|           | eine Zahlung erstellt oder bearbeitet wird, |  |
|      |         |**DANN**|           | darf der Wert für `DealerName` nicht leer sein. | |
|**B2**| Payment |        |           | | |
|      |         |**WENN**|           | eine Zahlung erstellt oder bearbeitet wird, |  |
|      |         |**DANN**|           | darf der Wert für `DealerLocation` nicht leer sein. | |
|**B3**| Payment |        |           | | |
|      |         |**WENN**|           | eine Zahlung erstellt oder bearbeitet wird, |  |
|      |         |**DANN**|           | muss das aktuelle Datum im gleichen Monat wie `TurnoverTime` sein. | |

#### Prüfung der Kreditkarten-Nummer

Prüfen Sie beim Einfügen oder beim Bearbeiten folgende Einschränkungen der Kreditkarten-Nummer. Der Algorithmus ist wie folgt definiert:

- Die Kreditkartennummer (16 Stellen) ist durch eine Prüfziffer (16. Stelle, die bei der Berechnung nicht mitverwendet wird) wie folgt gegen Fehleingaben abgesichert:  
- Die Ziffern an den Stellen mit gerader Nummer (beginnend bei 0) werden verdoppelt und deren Ziffernsumme wird aufsummiert  
- Die Ziffern an den Stellen mit ungerader Nummer werden aufsummiert  
- Die beiden erhaltenen Zahlen werden addiert und die Differenz zur nächsten Zehnerzahl wird ermittelt, die gleich groß oder größer ist als die Summe. Diese Ziffer (0-9) ist die Prüfziffer  

Ein Beispiel für die Kreditkartennummer 2718 2818 2845 8567:

| 4   |     | 2   |     | 4   |     | 2   |     | 4   |     | 8   |     | 16  |     | 12  |     |    | 34 |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|----|----|
|**2**|**7**|**1**|**8**|**2**|**8**|**1**|**8**|**2**|**8**|**4**|**5**|**8**|**5**|**6**|**7**|    |    |
|     | 7   |     | 8   |     | 8   |     |  8  |     |  8  |     | 5   |     | 5   |     |     | +  | 49 |
|     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     | =  | 83 |
|     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |====|====|
|     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |    | 7  |

> **HINWEIS:** Wenn die Kreditkarten-Nummer nicht korrekt ist, **muss eine Ausnahme geworfen werden.**

## Datenimport

Erstellen Sie ein Konsolenprogramm welches die Datenbank erzeugt und die beigelegte csv-Datei in die Datenbank importiert. Falls es beim Import zu Fehlern kommt (z.B. Kreditkarten-Nummer falsch), muss eine entsprechende Fehlermeldung ausgegeben werden.

---

### RESTful-Service  

Erstellen Sie einen REST-Service Zugriff für die Entität ***'Payment'*** mit folgende Komponenten:

- Ein ***Model*** für die Entität ***'Payment'***.
- Einen ***Kontroller*** mit den folgenden Operationen
  - Abfrage alle Zahlungen
  - Abfrage einer Zahlung mit der Id
  - Erstellen einer Zahlung (CardNumber, ExecutionTime, TurnoverTime, DealerName, DealerLocation, Turnover, Note)
  - Änderung einer Zahlung (CardNumber, ExecutionTime, TurnoverTime, DealerName, DealerLocation, Turnover, Note)
  - Löschen einer Zahlung (nur im gleichen Monat zulässig).

Prüfen Sie mit dem Werkzeug Swagger oder Postman die REST-Schnittstelle.

### MVVM-Fontend  
  
Erstellen Sie für die folgenden Entität Formulare im MVVM-Projekt:  
  
- List   - Übersicht der Zahlungen  
- Create - Erstellen einer Zahlung  
- Edit   - Bearbeiten einer Zahlung  
- Delete - Löschen eines Zahlung mit Rückfrage  

**Viel Spaß beim Umsetzen der Aufgabe!**
