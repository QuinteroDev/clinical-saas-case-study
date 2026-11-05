# Estudio de caso: SaaS clínico multihospital con extracción de datos por LLM

> Proyecto real en producción para un cliente bajo NDA. Aquí no hay código, datos ni nombres; solo el problema, las decisiones y cómo está construido.

## Contexto

Plataforma web de registro clínico para servicios de urología oncológica de varios hospitales. Los médicos registran pacientes de 9 patologías, consultan su evolución y exportan datos para investigación. Lo desarrollo y mantengo yo: backend, base de datos, despliegue y soporte.

## Qué he construido

- **Lectura automática de informes clínicos.** El médico sube un informe (anatomía patológica, imagen, biopsia), incluso un PDF escaneado, y el formulario se rellena solo. El LLM solo extrae lo que dice el texto; las decisiones clínicas (estadio TNM, grados) las calcula código determinista con tests. El nombre del paciente se anonimiza antes de llamar al modelo. Elegí el modelo comparando varios sobre cientos de informes reales, coste frente a precisión.
- **Buscador en lenguaje natural.** El médico pregunta en castellano y el LLM devuelve un JSON de filtros, que se valida contra un catálogo de campos permitidos antes de tocar la base de datos.
- **Ingesta de datos de otros hospitales con humano en el bucle.** Excel/CSV heterogéneos: anonimización sin IA, clasificación y emparejamiento de columnas con el LLM, validación por el equipo. Cola asíncrona con registro de tokens y coste por llamada.
- **Permisos multihospital.** Una sola regla por rol y hospital que usan los 72 permisos de módulo, la API y las descargas, cubierta con una matriz de tests de todas las combinaciones.
- **API REST con FHIR** para integraciones con otros sistemas sanitarios.
- **Unificación de 9 módulos clínicos** sobre piezas comunes, con tests que fallan si reaparece código duplicado.

## Decisiones

- Orquestación de LLM escrita a mano sobre el SDK de OpenAI, sin frameworks: el flujo es corto y así controlo cada llamada.
- La IA propone y Python decide en todo lo que tenga consecuencias clínicas.
- Correcciones de datos en producción en dos pasos: primero solo informar, luego corregir con copia previa.

## Stack

Python · Django · Django REST Framework · PostgreSQL · OpenAI API (texto y visión) · HL7 FHIR · pytest (~2.950 tests) · AWS EC2 · Amazon SES · Nginx · Gunicorn
