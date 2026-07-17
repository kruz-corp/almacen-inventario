[documentacion.md](https://github.com/user-attachments/files/30125476/documentacion.md)
# Almacén·Inventario — Documentación del producto

**Estado:** Prototipo funcional (MVP) validado con 2 empresas demo de rubros distintos.
**Última actualización:** Julio 2026

---

## 1. Qué es esto

Sistema de control de inventario **multi-cliente**: un único código (`index.html`) sirve para cualquier empresa que se dé de alta, sin necesidad de tocar el código para cada cliente nuevo. Cada empresa ve únicamente sus propios datos (productos, usuarios, sucursales), aislados del resto por reglas de seguridad a nivel de base de datos.

Construido como aplicación de una sola página (HTML/CSS/JS), sin backend propio — toda la lógica de datos y seguridad vive en Supabase (Postgres + Auth + Row Level Security).

---

## 2. Infraestructura

| Recurso | Detalle |
|---|---|
| Repositorio | `github.com/kruz-corp/almacen-inventario` (público, para permitir GitHub Pages gratis) |
| Hosting | GitHub Pages — `https://kruz-corp.github.io/almacen-inventario/` |
| Base de datos / Auth | Supabase (plan Free), proyecto `almacen-inventario` |
| Cuenta | Correo separado (`guevara.kruz@gmail.com`), independiente de MARTIZENT |

**Archivos del repo:**
- `index.html` — la app que usan los clientes (login → catálogo de su empresa)
- `admin.html` — panel de uso exclusivo del superadmin, para dar de alta empresas nuevas sin escribir SQL

---

## 3. Modelo de datos (core reutilizable)

Estas tablas son compartidas por todos los módulos futuros (inventario, ventas, etc. — hoy solo existe inventario):

- **`empresas`** — cada cliente que compra el producto
- **`sucursales`** — sucursales de cada empresa
- **`roles`** — roles definidos por empresa (ej. "Administrador", "Operador")
- **`permisos_modulo`** — qué puede hacer cada rol en cada módulo (`puede_ver`, `puede_editar`, `puede_eliminar`)
- **`usuarios`** — usuarios de cada empresa, vinculados a su cuenta real de login vía `auth_user_id`
- **`empresa_config`** — qué módulos tiene activos cada empresa
- **`catalogo_productos`** — específica del módulo inventario (productos, stock, precios)
- **`superadmins`** — lista de emails con permiso para crear empresas nuevas (solo vos, por ahora)

**Seguridad (RLS):** cada tabla tiene reglas que verifican que el usuario logueado solo pueda ver/editar datos de su propia empresa (`empresa_id = auth_empresa_id()`), y que las acciones de edición/eliminación respeten el permiso de su rol. El superadmin tiene reglas adicionales que le permiten operar sobre cualquier empresa.

---

## 4. Funcionalidad actual del módulo Inventario

- Login con email/contraseña (Supabase Auth)
- Ver catálogo de productos con buscador y filtro por categoría
- Estadísticas: productos activos, productos con stock bajo, valor total de inventario
- Agregar producto nuevo
- Editar producto existente
- Eliminar producto (según permiso del rol)
- Ajustar stock rápido (+/- desde la tabla)
- Exportar catálogo a Excel
- Importar catálogo desde Excel, con **bloqueo automático si detecta duplicados** (por código o nombre, contra el catálogo existente o dentro del mismo archivo)
- Permisos diferenciados por rol (ver / editar / eliminar)

---

## 5. Cómo dar de alta un cliente nuevo (paso a paso)

1. **Crear su cuenta de acceso** en Supabase → Authentication → Users → Add user (email + contraseña provisoria, tildando "Auto Confirm User")
2. Copiar el **UUID** que Supabase le asigna
3. Entrar a `admin.html` con tu cuenta de superadmin
4. Completar el formulario **"Nueva empresa cliente"**: nombre, rubro, nombre y email del administrador, y pegar el UUID
5. Esto crea automáticamente: la empresa, sus 2 roles por defecto (Administrador / Operador) con permisos ya configurados, y vincula al primer usuario
6. Pasarle al cliente la URL (`https://kruz-corp.github.io/almacen-inventario/`) y sus credenciales
7. El cliente entra con catálogo vacío y lo carga solo (manualmente o importando un Excel)

Para sumar un **segundo usuario** a una empresa que ya existe, se usa la segunda sección de `admin.html` ("Agregar usuario a empresa existente"), mismo proceso pero sin recrear la empresa.

---

## 6. Empresas demo actualmente cargadas (para pruebas)

| Empresa | Rubro | Usuario admin | Usuario secundario |
|---|---|---|---|
| Ferretería Demo S.R.L. | Ferretería | admin@ferreteriademo.test | juan@ferreteriademo.test (Encargado de inventario) |
| Farmacia Demo S.R.L. | Farmacia | admin@farmaciademo.test | maria@farmaciademo.test (Farmacéutico) |

---

## 7. Limitaciones conocidas (a resolver antes de vender esto en serio)

- El alta de usuarios sigue requiriendo un paso manual en Supabase (crear la cuenta de Auth y copiar el UUID) — no está 100% automatizado desde el panel
- Sin recuperación de contraseña self-service para los clientes (hoy la resetearías vos manualmente desde Supabase si alguien la olvida)
- Sin módulos más allá de inventario todavía (ventas, campo, etc. quedan pendientes)
- Sin facturación/cobro automatizado — hoy es 100% manual
- El plan Free de Supabase tiene límites (pausas por inactividad, límite de filas/requests) — antes de tener clientes reales pagando, hay que evaluar pasar a un plan pago

---

## 8. Próximos pasos posibles

- Definir modelo de negocio y pricing (ver documento aparte / conversación)
- Evaluar automatizar la creación de usuarios de Auth desde el panel admin (requiere backend propio o función serverless, ya que el frontend no puede usar la service_role key)
- Sumar un segundo módulo (ej. ventas/campo) reusando el mismo core
- Recuperación de contraseña self-service
- Migrar a plan pago de Supabase cuando haya clientes reales
