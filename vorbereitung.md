# Klausur vorbereitung

### Rest - Representational State Transfer
- erlaubt JSON / XML
- ist Ressourcenorientiert
- RESTful Anwendungen unterstützen CRUD

### Backend Schichten und eingesetzte Frameworks
- Entitäts-Layer	-   model	- Mongoose
- Control-Layer	-   Service - Node.js
- Boundary-Layer	-   Router	- Express
### MognoDB - NoSQL-DBMS
- Collections
- (JSON) Documents
- Paths bzw “Properties”
### Mongoose
- ODM		Object-Document-Mapper
- Definition über Schemas
- Datenbank-Dokumente werden Objekte
### Schema
- nur in Mongoose und bezieht sich auf eine MongoDB-Collection
- wir erzeugen Interface für Typisierung

#### Aufbau
```
export interface INameCollection{
	name: String
	public?: boolean;
}
```
