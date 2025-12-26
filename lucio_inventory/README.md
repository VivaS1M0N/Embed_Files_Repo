# Inventario de Patio Covers (lucio_inventory)

## Cómo probarlo en local
1. Abre `inventory_manager_lucio.html` directamente en el navegador (doble clic o arrastrando el archivo). No requiere build ni dependencias; el documento ya incluye React, Tailwind y Babel vía CDN.
2. Usa la pestaña **Sobrantes de Aluminio** para cargar retazos. Pega un JSON con nuevas categorías/ítems en el cuadro de importación (formato: `{ "Nombre Categoria": ["Item 1", "Item 2"] }`). El catálogo se fusiona sin duplicados y se ajusta automáticamente el formulario.
3. Usa **Descargar Excel (.csv)** para exportar tanto sobrantes como consumibles.

## Cómo publicar en Amazon Web Services (estático)
1. **Bucket S3**: crea un bucket público para hosting estático y sube `inventory_manager_lucio.html` junto con los PDFs de referencia si los necesitas junto al archivo.
2. **CloudFront (opcional)**: crea una distribución que apunte al bucket para HTTPS y caching.
3. **Control de versiones**: cada vez que actualices el archivo, vuelve a subirlo al bucket; CloudFront invalidará el objeto si usas la distribución.

## Sistema de usuarios en AWS (sugerido)
- Usa **Amazon Cognito User Pools** para registro/inicio de sesión. Configura un App Client con flujo de usuario-password.
- Protege el HTML detrás de CloudFront agregando una función Lambda@Edge que verifique el token `id_token` de Cognito o coloca el HTML detrás de un **Application Load Balancer** con autenticación de Cognito habilitada.
- Una alternativa sin Edge es servir la página desde una pequeña app en **AWS Amplify Hosting** usando el mismo HTML; Amplify ofrece integración directa con Cognito.

## Base de datos SQL en AWS
- Reutiliza tu instancia SQL existente (RDS o EC2). Para integrar:
  1) Despliega una pequeña API (AWS Lambda + API Gateway o una app en ECS/EC2) que exponga endpoints REST/GraphQL para leer/escribir inventario.
  2) Esa API debe manejar credenciales de la BD (usa AWS Secrets Manager o Parameter Store) y validar tokens de Cognito.
  3) Desde `inventory_manager_lucio.html`, realiza `fetch` a los endpoints; al usar Cognito agrega el header `Authorization: Bearer <id_token>`.
- Tabla mínima recomendada: `aluminum_items(id, category, name, color, length_ft, length_in, cut, qty, location)` y `base_items(id, name, category, min_stock, current_stock, unit)`.

## Formato para extraer categorías desde PDFs
- Exporta del PDF una lista en texto y conviértela a JSON con la forma `{ "Categoria": ["Item 1", "Item 2"] }`.
- Pega ese JSON en el campo de importación de la pestaña **Sobrantes de Aluminio** para crear nuevas categorías sin editar el código.

## Dependencias
- Todo se sirve desde CDN; no hay instalación adicional.

