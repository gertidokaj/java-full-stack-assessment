# java-full-stack-assessment

Secured REST API for satellite imagery product discovery (GeoJSON AOI), plus a web client with map, OAuth2/Keycloak login, and OpenAPI docs.

---

## How to run

### What you need

- Java 17 or higher (check with `java -version`)
- Maven, or use the included wrapper (`./mvnw` / `mvnw.cmd`)
- For Keycloak: Docker and Docker Compose. On Windows, Docker Desktop must be installed and running.

### Step 1: Keycloak (optional)

The app works with or without Keycloak. If you skip this, you'll use a simple username/password form instead.

1. Open a terminal in the project folder.
2. Run:
   ```bash
   docker compose up -d
   ```
   On Windows, use Command Prompt or PowerShell. Make sure Docker Desktop is running first.
3. Wait ~30 seconds for Keycloak to start.
4. Keycloak runs at http://localhost:8180. The realm `geocento` and user `user` are imported from `keycloak/realm-export.json`. If something goes wrong, you can add the user manually: open http://localhost:8180, log in as admin/admin, go to Realm geocento → Users → Add user (username: user, password: password).

If you don't run Keycloak: the app will show "Keycloak not running" and a username/password form after a few seconds. Use user / password.

### Credentials

| Use | Username | Password |
|-----|----------|----------|
| App login | user | password |
| Keycloak admin (http://localhost:8180) | admin | admin |

### Step 2: Start the app

1. In the project folder, run:
   ```bash
   ./mvnw spring-boot:run
   ```
   On Windows:
   ```bash
   mvnw.cmd spring-boot:run
   ```
2. Wait until you see something like "Started JavaFullStackAssessmentApplication" in the console.
3. The app is at http://localhost:8080.

### Step 3: Use the web client

1. Open http://localhost:8080 in a browser.
2. Sign in:
   - With Keycloak: click "Sign in with Keycloak", then enter user / password on the Keycloak page.
   - Without Keycloak: use the form on the app page with user / password.
3. After login you see the map and a sidebar.

### Step 4: Search for products

- Draw a polygon: click the pentagon icon in the map toolbar, then click on the map to draw. Close the shape by clicking the first point again.
- Or paste GeoJSON: copy a Polygon into the text area. There's a "Paste example" link that fills in a working example.
- Click "Search products". Results show in the sidebar and as coloured shapes on the map.

### Example GeoJSON to paste

This one finds the Sentinel-2A product (UK area):

```json
{
  "type": "Polygon",
  "coordinates": [[[-0.5, 51.0], [0.5, 51.0], [0.5, 52.0], [-0.5, 52.0], [-0.5, 51.0]]]
}
```

### If something goes wrong

- "Connection refused" on port 8180: Keycloak isn't running. Either start it with `docker compose up -d` or wait for the fallback form.
- "Connection refused" on port 8080: the app isn't running. Run `./mvnw spring-boot:run` again.
- Stuck on "Checking...": wait a few seconds. If Keycloak is down, the form should appear.
- No products: try the example polygon above. The mock products cover UK, Alps, and France.

---

## API

- GET `/api/health` — no auth, returns "API is running".
- POST `/api/products/search` — needs auth. Send a GeoJSON Polygon in the body. Returns a GeoJSON FeatureCollection. Auth: Bearer token (from Keycloak) or HTTP Basic (user/password). In Swagger UI, use Authorize to set the token.

## Swagger / OpenAPI

- Swagger UI: http://localhost:8080/swagger-ui.html
- OpenAPI JSON: http://localhost:8080/v3/api-docs

To test with a token: log in via the web app, get a token from the browser (or Keycloak), then in Swagger click Authorize and paste `Bearer <token>`.

## Tests

```bash
./mvnw test
```

Keycloak doesn't need to be running; tests use a mock JWT decoder.

---

## GOAL

Write a secured service exposing REST API endpoints for satellite imagery product discovery:
- Design endpoints that accept Area of Interest using GeoJSON format
- Response should include satellite imagery products with their footprints and metadata in GeoJSON format
- Products should only be returned if they spatially intersect with the provided AOI

In addition, write a simple web client displaying a map. When the application is loaded it should ask the user to authenticate. Once authenticated, prompt the user to provide an AOI (Area of interest) in GeoJSON format or by drawing on the map. Use this AOI to call the API and display the returned product footprints on the map along with product information.

### IMPORTANT
- Design your own API structure - consider RESTful principles, HTTP methods, and proper response formats
- The API needs to be secured using a token obtained from a service (Keycloak or other OAuth based services), ideally using a User account
- The actual backend is a mock, no need for a DB. The list of products returned can be anything as long as they "make sense" geometrically.
- Display product information (name, acquisition date, provider, resolution) alongside the map footprints
- Include product thumbnails in the display (placeholder images are fine)
- Use Maven

### BONUS
- Generate the OpenAPI specs and doc page from the code
- Advanced AOI selection tools (drawing on map vs text input)

## PRODUCT DATA STRUCTURE (SAMPLE)

Your mock products should include below satellite imagery attributes:
```json
{
  "id": "product-id",
  "name": "Sentinel-2A",
  "acquisitionDate": "2024-01-15T10:30:00Z",
  "provider": "ESA",
  "resolution": "10m",
  "thumbnailUrl": "/thumbnails/product1.jpg",
  "metadata": {
    "sensor": "MSI",
    "cloudCoverage": 15
  },
  "footprint": {
    "type": "Polygon",
    "coordinates": [[[lon,lat], [lon,lat], ...]]
  }
}
```

## UI
The web client uses [Tabler](https://tabler.io) (MIT): https://github.com/tabler/tabler

## TIPS and links
- https://geojson.org/
- if you decide to use Keycloak https://www.keycloak.org/ also deploy a Keycloak instance using docker compose https://www.keycloak.org/server/containers
- leaflet display of geojson https://leafletjs.com/examples/geojson/
- Mock satellite data: Create realistic product names (Sentinel-2, Landsat-8, SPOT, etc.)
- Simple spatial intersection: Bounding box overlap is acceptable for mock implementation

## EVALUATION CRITERIA
The work will be evaluated against the following criteria
- the evaluator should be able to run a working solution following the README instructions
- quality of the code and good comments
- some unit testing (no need to have full code coverage, only to demonstrate the ability to write tests)
- API design demonstrates understanding of RESTful principles
- Presentation of product information with thumbnails
- Map integration shows footprints correctly linked to product information
