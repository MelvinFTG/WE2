# Klausur vorbereitung
## Inhalt
1. [Backend](#backend)
    - [Rest](#rest)
    - [HTTP](#http)
    - [Mongoose](#mongodb--mongoose)
    - [Express](#express)
    - [Sicherheit](#sicherheit)
2. [Frontend](#frontend)
# Backend
## Rest
#### Representational State Transfer
- erlaubt JSON / XML
- ist Ressourcenorientiert
- RESTful Anwendungen unterstützen CRUD

#### Rest Constraints
- Client-Server:
    Client schickt **Requests** an Server, Server sendet **Response** an Client.
- Stateless Communication:
    Jeder **Request muss selbstbeschreibend** sein.
- Cache:
    Daten müssen gekennzeichnet werden können <sub>z.B. _cachaeble_ bzw. _non-cachable_</sub>.
- Layered System:
    Komponenten sind in Layern organisiert.
- Uniform Interface:
    > its emphasis on a uniform interface between components

### Resource
- eine Resource kann über einen **Hyperlink (URI)** referenziert werden
- eine Resource kann auch das **Ergebnis einer Berechnung** sein
- eine Resource kann **verschiedene Repräsentationen** haben

- Beispiele:
    - Textdokument, Bild
    - Wetter in Berlin <sub> zum Zeitpunkt der Abfrage </sub>
    - Summe zweier Zahlen 
    - Collection <sub>andere Resources</sub>

### Uniform Interface
- zentrales Merkmal von REST und hat volgende Eigenschaften
    - die Resources werden über **Bezeichner (URI)** identifiziert
    - die Resources werden über die **Repräsentation (bspw. JSON)** manipuliert
    - Selbst-beschreibende Messages

## HTTP
### HTTP Requests
- **Target:** eine URI
- **Methode:** was de Sender will
- **Header-Fields:** Felder mit Informationen <sub>Modifizierer, Metadaten, Client Informationen </sub>
- **Body:** der eigentliche Content, meist die Resource

### HTTP Response
- **Status Code:** Code der das Ergebnis beschreibt <sub>z.B. 404</sub>
- **Header:** Felder mit Infos über Response
- **Body:** Content je nach Statuscode <sub>bspw. _Resource_ oder _Fehler Beschreibung_</sub>

### Request Methoden
|Methode|Beschreibung|
|---|---|
|GET| Selektierte Methode soll übertragen werden |
|POST| zum anlegen einer Resource <sub> Resource wird vom Sender mitgeschickt</sub> |
|PUT| Resource soll verändert werden |
|DELETE| Selektierte Resource soll gelöscht werden |

### Response Status Codes
|Aussage| Status Code | Beispiel |
|---|---|---|
| Informational | 1xx | |
| Successful | 2xx |    **200:** _OK_ - **201:** _Created_ - **204:** _No Content_ -- bei Delete |
| Redirection | 3xx | |
| Client Error | 4xx |  **400:** _Bad Request_ -- Anfrage fehlerhaft - **401:** _Unauthorized_ -- Client nicht berechtigt - **404:** _Not Found_ -- Resource gibt es nicht |
| Server Error | 5xx |  **500:** _Internal Server Error_ -- generischer Fehler - **501:** _Not Implemented_ -- Aktion wird nicht unterstützt |

### Backend Schichten und eingesetzte Frameworks
| **Layer** | _module_| **Framework**|
| :---|---|---|
|Entitäts-Layer	|   mongoose-model	| Mongoose|
|Control-Layer	|   Service | Node.js|
|Boundary-Layer	|   Router	| Express|

## MongoDB / Mongoose
### MognoDB - NoSQL-DBMS
- Collections
- (JSON) Documents
- Paths bzw “Properties”

### Mongoose
- ODM - _Object-Document-Mapper_
- Definition über Schemas
- Datenbank-Dokumente werden Objekte

### Schema
- nur in Mongoose und bezieht sich auf eine MongoDB-Collection
- wir erzeugen Interface für Typisierung
- Mongoose generiert eine Klasse 

#### Code Beispiel Schema
```typescript
export interface INameCollection{   // Dokument-Typinformation
    owner: Types.ObjectId
    name: string
    description?: string
    public?: boolean;
}

const nameCollectionSchema = new Schema<INameCollection> ({ // Schema
    owner: {type: Schema.Types.ObjectId, ref: "User", required true },
    name: { type: String, required: true },
    description: String,
    public: { type: Boolean, default: false }
})

export const NameCollection = model("NameCollection", nameCollectionSchema);    // Klasse um auf Dokument zuzugreifen
```

### Middleware in Mongoose
- auch _hooks_ genannt wird im Model definiert
- wird angemeldet _pre_ bzw. _post_ bestimmten Aktionen
- daten werden über **this** übergeben
    - **Document:** bei _save_ zeigt **this** auf das [Document](#code-beispiel-pre-hook-auf-document)
    - **Query:** bei _updateMany_ zeigt **this** auf die [Query](#code-beispiel-pre-hook-auf-query)

#### Code Beispiel _pre_ hook auf **Document**
```typescript
userSchema.pre("save", { document: true, query: false }, async function () {
    logger.info("fired save document");

    if (this.isModified("password")) {
        const pwdHash = await bcrypt.hash(this.password, 10);
        this.password = pwdHash;
        logger.debug("user password hashed");
    }
})
```

#### Code Beispiel _pre_ hook auf **Query**
```typescript
userSchema.pre("updateMany", { document: false, query: true }, async function () {
    logger.info("fired update many querry")

    const update = this.getUpdate() as Query<any, IUser> & { password?: string } | null;
    if (update && update.password) {
        this.set({ password: await bcrypt.hash(update.password, 10) });
        logger.debug("user password hashed");
    }
    else logger.info("no password change")
})
```
## Express
### Was ist Express
- JavaScript Framework für Entwicklung von Backends
- Routing genanntes Mapping von Requests

#### Code Beispiel Express
```typescript
import express from "express"; // Express-Framework
import http from "http"; // Nodejs Build-in

const app = express(); // Erzeugung der App
          //URI               //Funktion an die der Request geleitet wird
app.get('/hallo', (req, res) => { res.send({ msg: 'Hello World!' }); }); // Anmelden einer Route
//Request Methode                           //Das Ergebnis
```

### Router
- Für jede Request-Methode gibt es Methoden zum Anmelden des Handlers
    - get
    - post
    - put
    - delete
#### Code Beispiel app.get
```typescript
app.get('/some/path', (req, res) => { ... } );
```
#### Code Beispiel Router-Hierarchie
```typescript
app.use("/user", userRouter);

const userRouter = express.Router();
userRouter.get("/count", (req, res) => {
…
})

// Gesamtpfad: /user/count
```

### Express Request
- **Content** über body zugreifbar: field _body_
- **Header** kann man über get lesen: field _header_
- **Parameter** sind platzhalter: field _params_
- **Query** Argumente liegen als Objekte vor: field _query_

### Express Response
- finden sich in _express.Response_ wieder
- **Content** auch hier im _body_
- **Header** kann über _set()_ setzen oder _getHeader()_ lesen und schreiben
- **Status-Code** üder _status()_ oder _sendStatus()_ gesetzt bei _setStatus()_ mit Text im Body automatisch

### Request Validierung
- Eingabe niemals trauen <sub>jeder kann einen Request beliebig
manipulieren</sub>
- validierung im Router hat den Vorteil, dass unnötige Datenbankabfrage verhindert wird
- Validierung über express-validator als middleware gibt JSON Fehler zurück

### Sanitization
- Daten _reinigen_
- Prüfung der Länge
- Normalisierung z.B. _E-Mail-Adressen_
- Ersetzen von _problematischen_ Zeichen
- Löschen von nicht benötigten Daten

## Sicherheit
### Schutzziel: CIA
- Confidentiality
- Integrity
- Availability

- werden u.a. mit Middlewares umgesetzt

### Authentifizierung und Autorisierung
- **authentication:** Verifizierung der behaupteten Identität einer Entität
- **authorization:** Einräumung von Rechten, inklusive der Gewährung von Zugriff aufgrund von Zugriffsrechten

### Verifikation
Kann auf verschiedene Verfahren durchgeführt werden, die auf verschiedenen **Faktoren** basieren
- **Wissen:** Passwörter, PIN
- **Besitz:** Schlüssel, Karte, Telefon
- **Inhärenz:** Biometrie

### Asymmetrische Verschlüsselung
|private.key|public.crt|
|:---:|:---:|
| openssl -rsoutl **-sign** -inkey **private.key** -in name.txt -out **text.sgn** | openssl rsautl **-encrypt** -inkey **public.crt** -certin -in text.txt -out **text.enc**|
|openssl rsautl **-decrypt** -inkey **private.key** -in **text.enc** -out text.dec| openssl rsautl **-verify** -inkey **public.crt** -certin -in **text.sgn** -out text.vfy|

### JSON Web Token (JWT)
- **Bearer-Token** (Access-Token)
    - Access Token sind Credentials
    - wird wie Passwort verwendet um auf geschützte Ressourcen zuzugreifen
    - der String enthält auf Client ausgestellte Autorisation
- **JWT**
    - Standard für Web-Apps 
    - kompaktes Darstellungsformat
    - eingesetzt bei HTTP Authorization

    - Base-64 dekodiert (bzw. Base64URL)

# Frontend
### Webpack
- static assets oder css Datein werden automatisch eingebunden
- werden im JavaScript Quelltext importiert
- man benötigt kein asset verzeichnis
~~~typescript
import React from 'react';      // normaler Modul Import
import logo from './logo.svg';  // 'logo' kann als Pfandangabe verwendet werden
import './App.css';             // eine CSS Datei
~~~

## React
- vereinfacht Arbeit mit DOM
- ausgelegt auf Single Page Apps 
- extrem performant

### UI-Komponenten
- gibt 2 Arten von Komponenten
    - **Class Components:** überschreibt die Methode _Render_
    ~~~typescript
    export class ClassComp extens React.Component{
        override render(): JSX.Element {
            return <div>Hello World</div>;
        }
    }
    ~~~
    - **Function Components:** ganz normale JavaScript-Funktionen
    ~~~typescript
    export function FunctionComp(){
        return <div>Hello world</div>;
    }
    ~~~

### JSX
- syntactic sugar für React-Elemente
- HTML ähnliche Syntax
- **Keine** DOM-Elemente/-Nodes **sondern** React-Elemente/-nodes