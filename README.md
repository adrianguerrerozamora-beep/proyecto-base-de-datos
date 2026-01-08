# EventFlow: Sistema de Gestión Integral para DJ

**EventFlow** es una plataforma web full-stack diseñada para la gestión profesional de eventos musicales. Este sistema centraliza la logística, la venta de boletos y el control de acceso en tiempo real, resolviendo la problemática de la información dispersa que enfrentan los DJs independientes.

La aplicación permite administrar múltiples roles (Admin, Artista, Cliente), gestionar eventos públicos y privados, vender boletos mediante códigos QR únicos y validar el acceso en la puerta utilizando un escáner integrado o registro manual.

---

## 🚀 Características Principales

### 🎧 Para el DJ (Administrador)
- **Dashboard en Tiempo Real:** Panel de control con métricas en vivo sobre aforo y ventas.
- **Gestión de Eventos:** CRUD completo para eventos públicos y privados.
- **Control de Acceso (Live Manager):** Sistema de validación de entradas mediante escaneo de QR y registro rápido de pagos en puerta (Cover).
- **Gestión de Talentos:** Flujo de aprobación para solicitudes de nuevos artistas.

### 🎟️ Para el Cliente (Usuario)
- **Cartelera Digital:** Visualización de próximos eventos con detalles de lugar, fecha y lineup.
- **Compra de Boletos:** Flujo de compra seguro que genera un boleto digital único.
- **Código QR:** Acceso instantáneo a su código de entrada personal para mostrar en el evento.

### 🎤 Para el Artista
- **Perfil Profesional:** Registro con nombre artístico y solicitud de aprobación.
- **Agenda:** Visualización de los eventos en los que participa.

---

## 🛠️ Tecnologías Utilizadas

Este proyecto implementa una arquitectura moderna y escalable, seleccionada por su rendimiento y facilidad de despliegue:

- **Frontend & Fullstack Framework:** Next.js 14 (App Router) - Para una aplicación rápida y optimizada para SEO.
- **Base de Datos & BaaS:** Supabase (PostgreSQL) - Base de datos relacional robusta en la nube.
- **Lenguaje:** JavaScript (React).
- **Estilos:** CSS Modules con diseño "Neón/Dark Mode" responsivo.
- **Librerías Clave:**
    - `lucide-react`: Iconografía moderna y ligera.
    - `react-qr-code`: Generación dinámica de códigos QR en el cliente.
    - `@supabase/ssr`: Autenticación segura y manejo de sesiones en el servidor.

---

## 💾 Arquitectura de la Base de Datos

El núcleo de EventFlow es una base de datos relacional en PostgreSQL, diseñada siguiendo estrictamente las reglas de normalización para garantizar la integridad de los datos.

### 1. Modelo Entidad-Relación Extendido (EER)

El diseño conceptual utiliza **herencia y especialización** para manejar los diferentes actores del sistema de manera eficiente y evitar la redundancia de datos.

- **Jerarquía de Usuarios (Herencia Disjunta y Total):**
  Se implementó una superclase `Persona` que contiene los datos comunes (nombre, correo, contraseña). De ella heredan tres subclases:
  - **Cliente:** Hereda de Persona y agrega atributos específicos como historial de compras.
  - **Artista:** Hereda de Persona y agrega nombre artístico, género musical y estado de solicitud.
  - **Admin:** Hereda de Persona y posee privilegios de gestión total.

- **Entidades Débiles y Fuertes:**
  - `Evento`: Entidad fuerte principal que agrupa toda la logística.
  - `Playlist`: Modelada como entidad débil dependiente del Evento (una playlist no tiene sentido sin un evento asignado).



### 2. Modelo Relacional (Implementación Física)

La transformación al modelo relacional resultó en un esquema normalizado que garantiza la integridad referencial y el rendimiento.

- **Integridad Referencial:** Uso estricto de Claves Foráneas (FK) con reglas de eliminación en cascada (`ON DELETE CASCADE`) donde aplica. Por ejemplo, si se elimina un evento, automáticamente se eliminan sus boletos y registros de acceso para no dejar datos huérfanos.
- **Resolución de Relaciones N:M (Muchos a Muchos):**
  - `evento_artista`: Tabla intermedia creada para permitir que múltiples artistas participen en un solo evento y que un artista pueda participar en múltiples eventos a lo largo del tiempo.
  - `playlist_cancion`: Tabla intermedia para gestionar qué canciones pertenecen a qué playlist, permitiendo reutilizar canciones.



---

## ⚡ Consultas y Optimización (JOINs)

Para ofrecer una experiencia de usuario rápida y fluida, el sistema evita realizar múltiples peticiones pequeñas al servidor. En su lugar, utiliza consultas SQL complejas optimizadas directamente en el motor de base de datos.

### Uso Estratégico de JOINs
En lugar de traer el evento, luego buscar al cliente, y luego buscar a los artistas por separado, utilizamos `JOINs` para reconstruir la información completa en una sola transacción eficiente.

**Ejemplo Práctico en el Sistema:**
Al cargar la cartelera de eventos en la página principal, el sistema ejecuta una consulta que une 3 tablas simultáneamente:

1.  **Tabla `evento`:** Obtiene los datos base (título, lugar, fecha, precio).
2.  **LEFT JOIN `evento_artista`:** Conecta con la tabla intermedia para ver si hay artistas asignados.
3.  **LEFT JOIN `artista`:** Obtiene el "Nombre Artístico" real para mostrar el Lineup.
4.  **LEFT JOIN `cliente` (vía `persona`):** Si es un evento privado, obtiene el nombre del cliente que contrató el evento.

Esto permite que la tarjeta del evento muestre toda la información necesaria (incluyendo el lineup completo) en milisegundos.

---

## 🛡️ Seguridad y Protección de Datos

La seguridad fue una prioridad desde el diseño, implementando una estrategia de "Defensa en Profundidad" que protege la aplicación en múltiples niveles.

### 1. Row Level Security (RLS) en PostgreSQL
No confiamos solo en el código del frontend o backend. La seguridad está aplicada directamente en el motor de la base de datos mediante Políticas RLS:
- **Lectura Pública:** Cualquiera puede ver (`SELECT`) los eventos donde `es_publico = true`.
- **Privacidad de Datos:** Un usuario solo puede ver sus propios boletos. La base de datos verifica automáticamente que `auth.uid() == id_cliente`.
- **Escritura Restringida:** Solo los usuarios con rol `admin` tienen permiso para crear (`INSERT`), modificar (`UPDATE`) o eliminar eventos.

### 2. Prevención de SQL Injection
Al utilizar el cliente oficial de Supabase y ORM, todas las consultas son **parametrizadas automáticamente**. Los datos ingresados por el usuario nunca se concatenan directamente en la cadena SQL, neutralizando por completo los intentos de inyección de código malicioso.

### 3. Autenticación Robusta y Triggers
- **Gestión de Sesiones:** Uso de Cookies HttpOnly seguras y encriptadas.
- **Triggers de Base de Datos:** Cuando un usuario se registra, un disparador (`TRIGGER`) automático crea su registro en la tabla `persona` y le asigna el rol correspondiente, evitando errores humanos o manipulaciones en el proceso de registro.

---

## 📸 Vistazo al Proyecto

### 1. Página Principal (Cartelera)
Los usuarios pueden ver los eventos destacados y acceder a la compra de boletos.



### 2. Compra de Boletos & QR
Un modal interactivo permite confirmar la compra y entrega el QR al instante.



### 3. Panel de Control (DJ)
El DJ tiene herramientas para validar QRs y cobrar cover en la misma pantalla.



---

## 🔧 Instalación y Despliegue Local

Si deseas correr este proyecto en tu máquina local:

1.  **Clonar el repositorio:**
    git clone 
    cd eventflow

2.  **Instalar dependencias:**
    npm install

3.  **Configurar Variables de Entorno:**
    Crea un archivo `.env.local` en la raíz y agrega tus credenciales de Supabase:
    NEXT_PUBLIC_SUPABASE_URL=tu_url_de_supabase
    NEXT_PUBLIC_SUPABASE_ANON_KEY=tu_clave_anonima

4.  **Ejecutar el servidor de desarrollo:**
    npm run dev

5.  **Abrir en el navegador:**
    Visita http://localhost:3000

---

## 👨‍💻 Autor

Desarrollado por **Adrian Guerrero Zamora** como proyecto final para la materia de Base de Datos.