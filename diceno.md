# DISEÑO DE CLASES Y BASE DE DATOS - GRUPO 1 (Explorador de productos y mercado)

## 1. Diseño de Clases (DTOs, Modelos y Entidades)

### A. DTOs (Data Transfer Objects)
Son las clases que mapean exactamente el JSON que devuelven las APIs. No tienen lógica, solo datos.

**DummyJSON (Productos)**
*Nota: Esta API devuelve un objeto con una lista dentro, no un arreglo directo.*
- `ProductosResponseDto`: `products: List<ProductoDto>`
- `ProductoDto`:
  - `id`: Int
  - `title`: String
  - `price`: Double
  - `category`: String
  - `thumbnail`: String

**Frankfurter (Tipos de cambio)**
*Nota: Esta API devuelve un arreglo directo de tasas.*
- `RateDto`:
  - `date`: String
  - `base`: String
  - `quote`: String
  - `rate`: Double

**JSONPlaceholder (Publicaciones)**
*Nota: Esta API devuelve un arreglo directo.*
- `PostDto`:
  - `id`: Int
  - `title`: String
  - `body`: String

### B. Modelo del Dominio
Es la clase limpia que usará la UI, independiente de la API o la Base de Datos.
- `ProductoDominio`:
  - `id`: Int
  - `titulo`: String
  - `precio`: Double
  - `categoria`: String
  - `imagenUrl`: String

### C. Entidad de Room (Persistencia)
Es la clase que representa la tabla en SQLite.
- `ProductoFavoritoEntity`:
  - `id`: Int (`@PrimaryKey`)
  - `title`: String
  - `price`: Double
  - `category`: String

---

## 2. Diseño de la Base de Datos (Room)

**Tabla:** `productos_favoritos`

| Columna | Tipo SQLite | Tipo Kotlin | Restricciones |
| :--- | :--- | :--- | :--- |
| `id` | INTEGER | Int | **PRIMARY KEY** (No autoincremental, usamos el ID de la API) |
| `title` | TEXT | String | NOT NULL |
| `price` | REAL | Double | NOT NULL |
| `category` | TEXT | String | NOT NULL |

*Nota de diseño:* Se usa el `id` original de la API como clave primaria para evitar duplicados si el usuario guarda el mismo producto dos veces.

---

## 3. Capas de MVVM y su contenido

Nuestra arquitectura se divide en 3 capas principales:

1. **Capa de Datos (Model):**
   - *Remota:* `ProductoApiService`, `FrankfurterApiService`, `PostApiService` (Interfaces Retrofit).
   - *Local:* `AppDatabase`, `ProductoFavoritoDao` (Room).
   - *Repositorio:* `MarketRepository`. Decide si los datos vienen de internet o del almacén local. Expone funciones `suspend` para las APIs y `Flow` para la base de datos.
2. **Capa de Lógica de Negocio (ViewModel):**
   - `MarketViewModel`: Contiene el estado (`StateFlow` o `LiveData`) de la UI (productos, tasas, posts, favoritos, isLoading). Llama al Repositorio.
3. **Capa de UI (View):**
   - `MainScreen` (Compose): Lista de productos y buscador.
   - `DetailScreen` (Compose): Detalle del producto y botón de guardar.
   - `FavoritesScreen` (Compose): Lista de productos guardados y botón de eliminar.
   - `AppNavigation`: Gestiona las rutas entre pantallas.

---

## 4. Diagrama de Clases Detallado (Mermaid)

```mermaid
classDiagram
    %% Capa de Datos - Remota
    class ProductoApiService {
        <<interface>>
        +getProductos() ProductosResponseDto
    }
    class FrankfurterApiService {
        <<interface>>
        +getRates() List~RateDto~
    }
    class PostApiService {
        <<interface>>
        +getPosts() List~PostDto~
    }

    class ProductosResponseDto {
        +List~ProductoDto~ products
    }
    class ProductoDto {
        +Int id
        +String title
        +Double price
        +String category
        +String thumbnail
    }
    class RateDto {
        +String date
        +String base
        +String quote
        +Double rate
    }
    class PostDto {
        +Int id
        +String title
        +String body
    }

    %% Capa de Datos - Local (Room)
    class ProductoFavoritoEntity {
        <<Entity>>
        +Int id
        +String title
        +Double price
        +String category
    }
    class ProductoFavoritoDao {
        <<interface>>
        +getFavoritos() Flow~List~ProductoFavoritoEntity~~
        +insertFavorito(ProductoFavoritoEntity)
        +deleteFavorito(Int)
    }
    class AppDatabase {
        +productoFavoritoDao() ProductoFavoritoDao
    }

    %% Capa Repositorio
    class MarketRepository {
        -ProductoApiService productoApi
        -FrankfurterApiService rateApi
        -PostApiService postApi
        -ProductoFavoritoDao favoritoDao
        +obtenerProductos() List~ProductoDominio~
        +obtenerRates() List~RateDto~
        +obtenerPosts() List~PostDto~
        +obtenerFavoritos() Flow~List~ProductoFavoritoEntity~~
        +guardarFavorito(ProductoFavoritoEntity)
        +eliminarFavorito(Int)
    }

    %% Capa ViewModel
    class MarketViewModel {
        -MarketRepository repository
        +StateFlow~List~ProductoDominio~~ productos
        +StateFlow~Boolean~ isLoading
        +StateFlow~List~ProductoFavoritoEntity~~ favoritos
        +cargarProductos()
        +guardarFavorito(ProductoDominio)
        +eliminarFavorito(Int)
    }

    %% Relaciones
    ProductosResponseDto *-- ProductoDto : Contiene
    MarketRepository --> ProductoApiService
    MarketRepository --> FrankfurterApiService
    MarketRepository --> PostApiService
    MarketRepository --> ProductoFavoritoDao
    MarketViewModel --> MarketRepository
    AppDatabase --> ProductoFavoritoDao : Crea