# Configuración de Supabase para Urban Store

## Instalación y Configuración

### 1. Instalar Supabase CLI (opcional)
```bash
npm install -g supabase
```

### 2. Configurar Variables de Entorno
Crea un archivo `.env.local` en la raíz del proyecto con:

```env
NEXT_PUBLIC_SUPABASE_URL=tu_url_de_supabase
NEXT_PUBLIC_SUPABASE_ANON_KEY=tu_clave_anon_de_supabase
```

### 3. Estructura de la Base de Datos

Ejecuta estos comandos SQL en tu proyecto Supabase:

```sql
-- Tabla de colecciones
CREATE TABLE collections (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  visible BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Tabla de categorías
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  icon_url TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Tabla de productos
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  price NUMERIC(10,2) NOT NULL,
  images_url TEXT,
  category INTEGER REFERENCES categories(id),
  collection INTEGER REFERENCES collections(id),
  stock INTEGER DEFAULT 0,
  visible BOOLEAN DEFAULT TRUE,
  discount NUMERIC(5,2) DEFAULT 0,
  new BOOLEAN DEFAULT FALSE,
  favorite BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Tabla de pedidos
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  customer_name TEXT NOT NULL,
  customer_email TEXT NOT NULL,
  customer_phone TEXT,
  delivery_address TEXT NOT NULL,
  order_status TEXT DEFAULT 'pending',
  payment_status TEXT DEFAULT 'unpaid',
  payment_method TEXT,
  total_amount NUMERIC(10,2),
  discount_applied NUMERIC(5,2),
  delivery_date TIMESTAMP WITH TIME ZONE,
  notes TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Tabla de items de pedidos
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id UUID REFERENCES orders(id) ON DELETE CASCADE,
  product_id INTEGER REFERENCES products(id),
  quantity INTEGER NOT NULL,
  unit_price NUMERIC(10,2) NOT NULL,
  discount NUMERIC(5,2),
  total_price NUMERIC(10,2)
);

-- Índices para mejorar el rendimiento
CREATE INDEX idx_products_collection ON products(collection);
CREATE INDEX idx_products_visible ON products(visible);
CREATE INDEX idx_products_new ON products(new);
CREATE INDEX idx_products_favorite ON products(favorite);
CREATE INDEX idx_collections_visible ON collections(visible);
CREATE INDEX idx_orders_status ON orders(order_status);
```

### 4. Datos de Ejemplo

```sql
-- Insertar colecciones
INSERT INTO collections (name, description) VALUES
('Camisas', 'Camisas urbanas y modernas'),
('Hoodies', 'Sudaderas con capucha de alta calidad'),
('Gorras', 'Gorras y accesorios street style');

-- Insertar categorías
INSERT INTO categories (name, description) VALUES
('Ropa Superior', 'Camisas, hoodies y sudaderas'),
('Accesorios', 'Gorras y complementos'),
('Ropa Casual', 'Prendas para uso diario');

-- Insertar productos
INSERT INTO products (name, description, price, images_url, category, collection, stock, new, favorite) VALUES
('Camisa Urbana Negra', 'Camisa urbana de algodón premium', 29.99, '["/api/placeholder/400/500", "/api/placeholder/400/500"]', 1, 1, 50, true, false),
('Hoodie Minimalista', 'Hoodie de alta calidad con diseño minimalista', 49.99, '["/api/placeholder/400/500", "/api/placeholder/400/500", "/api/placeholder/400/500"]', 1, 2, 30, false, true),
('Gorra Street Style', 'Gorra ajustable con diseño urbano', 19.99, '["/api/placeholder/400/500"]', 2, 3, 100, true, true),
('Camisa Oversize', 'Camisa de corte amplio y cómodo', 34.99, '["/api/placeholder/400/500", "/api/placeholder/400/500"]', 1, 1, 25, false, false);
```

### 5. Activar las Páginas con Supabase

Para usar las páginas con integración de Supabase, renombra los archivos:

```bash
# Copiar las páginas actualizadas
cp src/app/page-supabase.tsx src/app/page.tsx
cp src/app/colecciones/page-supabase.tsx src/app/colecciones/page.tsx
cp src/app/pedidos/page-supabase.tsx src/app/pedidos/page.tsx
cp src/app/layout-supabase.tsx src/app/layout.tsx
```

### 6. Configuración de Políticas de Seguridad (RLS)

```sql
-- Habilitar RLS
ALTER TABLE collections ENABLE ROW LEVEL SECURITY;
ALTER TABLE products ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE order_items ENABLE ROW LEVEL SECURITY;

-- Políticas de lectura para usuarios anónimos
CREATE POLICY "Permitir lectura de colecciones visibles" ON collections
  FOR SELECT USING (visible = true);

CREATE POLICY "Permitir lectura de productos visibles" ON products
  FOR SELECT USING (visible = true);

CREATE POLICY "Permitir lectura de pedidos" ON orders
  FOR SELECT USING (true);

CREATE POLICY "Permitir lectura de items de pedidos" ON order_items
  FOR SELECT USING (true);

-- Políticas de inserción para pedidos
CREATE POLICY "Permitir creación de pedidos" ON orders
  FOR INSERT WITH CHECK (true);

CREATE POLICY "Permitir creación de items de pedidos" ON order_items
  FOR INSERT WITH CHECK (true);
```

### 7. Ejecutar el Proyecto

```bash
npm run dev
```

El proyecto estará disponible en http://localhost:8000

## Notas Importantes

- Los campos `visible`, `new` y `favorite` en la tabla `products` controlan qué productos se muestran y sus etiquetas.
- Los campos `visible` en la tabla `collections` controlan qué colecciones aparecen en el carrusel.
- El campo `images_url` debe ser un array JSON de URLs de imágenes.
- Para desarrollo local sin Supabase, el proyecto usará datos mock automáticamente.
