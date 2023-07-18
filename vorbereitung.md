# Klausur vorbereitung
## Inhalt
1. [Backend](#backend)
    - [Rest](#rest)
    - [HTTP](#http)
    - [Mongoose ](#mongodb--mongoose)
## Backend
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
```