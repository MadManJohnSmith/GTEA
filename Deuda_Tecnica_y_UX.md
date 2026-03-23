# Deuda Técnica y UX — GTEA
Última actualización: 2026-03-22

---

## Deuda de Seguridad

[CRÍTICO] Token de sesión almacenado en
  localStorage, no en HttpOnly cookie.
  El backend ya tiene CookieTokenAuthentication
  implementado y funcional, pero el frontend
  lo ignora completamente — AuthInterceptor y
  FacadeService leen de localStorage('gtea-
  proyecto-token'). Vulnerable a XSS.
  Archivo: interceptors/auth.interceptor.ts:27
           services/facade-service.ts:19

[CRÍTICO] Roles asignados por dominio de email.
  Cualquier persona puede registrarse como
  administrador usando email @admin.com.
  No hay validación de clave_admin en el flujo
  de registro del frontend más allá de enviarla
  al backend. Si el backend no la valida
  estrictamente, el sistema es comprometible.
  Archivo: services/facade-service.ts:152-200
           views/auth.py (registro no auditado)

[ALTO] AUTH_TOKEN_COOKIE_SECURE = False en
  authentication.py. Si se despliega en
  producción sin cambiar settings, el token
  viaja en HTTP plano.
  Archivo: authentication.py:43

[ALTO] Roles en CustomAuthToken — si un
  usuario tiene múltiples grupos Django, el rol
  asignado depende del orden no determinista
  de la DB. Comportamiento correcto solo
  garantizado si cada user tiene exactamente 1 grupo.
  Archivo: views/auth.py:47

---

## Deuda Arquitectónica

[ALTO] FacadeService es un god service de 266
  líneas que mezcla: auth HTTP, gestión de
  localStorage, validación de formularios,
  lógica de registro por rol, y utilidades de
  password. Viola single responsibility.
  Archivo: services/facade-service.ts

[ALTO] NuevoEventoWizard monolítico:
  485 líneas TS + 513 líneas HTML = 998 líneas
  totales para un solo flujo. Tres formularios
  reactivos inline, lógica de catálogos, lógica
  de edición, lógica de imágenes y lógica de
  conflictos mezcladas en un componente.
  Archivo: screens/admin/eventos/nuevo-evento-wizard/

[MEDIO] URLs de API sin versioning.
  /eventos/, /alumnos/, etc. No hay /api/v1/.
  Cualquier cambio de contrato rompe el frontend
  sin posibilidad de compatibilidad hacia atrás.
  Archivo: urls.py

[MEDIO] Dos package.json en el repo WebApp:
  raíz tiene ngx-cookie separado del package.json
  interno de Angular. Resolución de dependencias
  ambigua.

---

## Deuda de Integración

[CRÍTICO] EventoSerializer.inscritos declara
  IntegerField(read_only=True) pero ninguna
  view usa .annotate(inscritos=Count('inscripciones'))
  antes de serializar. El campo siempre retorna
  None o falla en runtime dependiendo de la
  versión de DRF.
  Archivo: serializers.py:97
           views/eventos.py (sin annotate)

[ALTO] EventosEdit.put() no usa EventoSerializer
  para validar datos en edición. Hace setattr()
  directo en el objeto modelo, bypasseando toda
  la validación de DRF. Un campo inválido se
  persiste sin error.
  Archivo: views/eventos.py:EventosEdit.put()

[ALTO] retrieveSignedUser() hace GET a
  /auth/login/ pero ese endpoint solo acepta
  POST (CustomAuthToken extiende ObtainAuthToken).
  Este método siempre retornará 405 Method Not
  Allowed.
  Archivo: services/facade-service.ts:100

[MEDIO] Wildcard imports en auth.py:
  'from GTEA_Project_API.serializers import *'
  'from GTEA_Project_API.models import *'
  Oculta dependencias reales, dificulta
  refactoring y puede traer colisiones de nombres.
  Archivo: views/auth.py:4-5

---

## Deuda de Testing

[ALTO] Cobertura real desconocida. Los archivos
  spec son boilerplate generado por Angular CLI.
  No hay evidencia de tests de integración API
  (pytest/DRF APITestCase) ni tests E2E.
  Sin tests, cualquier refactor es ciego.

[MEDIO] print() statements en código de
  producción:
  Logout: print("logout"), print(str(user))
  Archivo: views/auth.py:81-82

---

## Deuda de UX

[ALTO] hasConflict y conflictMessage declarados
  en NuevoEventoWizard pero nunca se actualizan.
  La UI tiene un bloque de alerta de conflicto
  de horario que NUNCA se muestra aunque exista
  conflicto. Feature muerta visible en el template.
  Archivo: nuevo-evento-wizard.ts:57-58
           nuevo-evento-wizard.html:~100

[MEDIO] setTimeout(() => {}, 0) para restaurar
  aulaId en modo edición. Hack de timing frágil
  que puede fallar si el render es más lento
  de lo esperado.
  Archivo: nuevo-evento-wizard.ts:_prefillForms

[MEDIO] EMAIL_DOMAIN_REGEX exportado de
  facade-service.ts pero no usado en ninguna
  validación de login visible. Código muerto
  o validación incompleta.
  Archivo: services/facade-service.ts:12

---

## Deuda Crítica de Integración y Deployment

~~[CRÍTICO — BLOQUEANTE] Peticiones HTTP comentadas en frontend.~~ ✅ RESUELTO 2026-03-23
~~Resuelto: todos los flujos implementados usan HTTP real.~~

~~[CRÍTICO — BLOQUEANTE] Docker/docker-compose.yml
  no existe en el repo.~~ ✅ RESUELTO 2026-03-23
  La entrega requiere
  deployment en la nube con Docker.
  Sin esto, no hay entrega posible.
  Archivo: No existe aún.

~~[CRÍTICO] Contratos de API no documentados
  formalmente.~~ ✅ RESUELTO 2026-03-23
  El consejo interno del equipo es
  "documentar qué endpoints y JSON espera tu código"
  pero no hay un contrato versionado. Si hay
  discrepancia entre lo que el frontend espera
  y lo que el backend entrega, la integración
  final falla sin punto de diagnóstico claro.

[CRÍTICO] EventoSerializer.inscritos sin annotate.
Resuelto parcialmente: is_full calculado correctamente.
inscritos sigue siendo ?? 0 — ver deuda pendiente.

~~[ALTO] Mocks sin equivalencia de contrato.~~ ✅ RESUELTO 2026-03-23
  Los mocks del frontend pueden tener estructura
  de datos diferente a lo que el backend realmente
  retorna. No hay validación cruzada hasta
  integración final.

## Deploy — Estado Actual (2026-03-23)
✅ Sistema desplegado en producción: https://gtea.ezarr.rocks
✅ Stack: nginx + Django (gunicorn) + MySQL en Docker Compose
✅ SSL/TLS activo con Let's Encrypt (certbot)
✅ HTTP activo y funcionando con backend real verificado manualmente
⚠️ HTTP/1.1 activo — HTTP/2 pendiente (cosmético, no bloqueante)

[ALTO — PENDIENTE TECH LEAD] inscritos en catálogo siempre muestra 0.
  Frontend usa ?? 0 como fallback defensivo hasta que el backend
  agregue .annotate(inscritos=Count('inscripciones__id')) en GET /eventos/.

[BAJO — post-entrega] estado 'inscrito' no se muestra en el catálogo.
  El backend no retorna estado de inscripción del alumno actual en
  GET /eventos/. Requiere endpoint adicional o campo extra en response.

[RESUELTO 2026-03-22] Flujo de inscripción alumno — end-to-end:
  POST /inscripciones/ con { evento_id } + token
  Backend deriva alumno desde request.user
  Frontend muestra toast éxito (201) y toast lista de espera (409)
  Botón deshabilitado reactivo vía signal isFull
  El estado correcto vive en evento-detalle (ya funcional).

~~MOCK_EVENTOS, MOCK_CATEGORIAS, MOCK_SEDES, MOCK_AULAS en evento-service.ts~~
~~Son arrays private readonly sin ningún consumidor. Eliminar en post-entrega.~~

[BAJO — post-entrega] editUser() en usuarios.ts:155 es TODO vacío.
  La edición de usuarios admin no está implementada.
  Archivo: screens/admin/usuarios/usuarios.ts:154

[BAJO — post-entrega] status en usuarios.ts:127 siempre es 'Activo'.
  El backend no expone este campo. Dato cosmético incorrecto.
  Archivo: screens/admin/usuarios/usuarios.ts:127
