# Checkpoint 4: Integraciones avanzadas e interconexión de sistemas

Workflow de n8n que conecta un agente con HubSpot (CRM), Gmail (casilla de soporte) y Slack (canal de operaciones) mediante OAuth2, con controles preventivos no-code.

## Archivos

- `checkpoint4_NahuelYair.json`: workflow exportado desde n8n (Workflow → Download).
- `docs/credentials.png`: captura de las credenciales Gmail, HubSpot y Slack conectadas.
- `Informe_Checkpoint4_Final_v4.pdf`: informe final con respuesta por ítem, evidencia y bitácora de intentos.

## Conectores

| Herramienta | Autenticación | Uso en el flujo |
|---|---|---|
| HubSpot (plan gratuito) | Managed OAuth2 | Búsqueda y creación de contactos |
| Gmail | Managed OAuth2 | Lectura de correos no leídos y creación de borradores |
| Slack | Managed OAuth2 | Publicación de avisos en el canal de operaciones |

En las tres credenciales, **Allowed HTTP Request Domains** está en `None`, de modo que no pueden llamar a otros dominios.

## Controles preventivos

1. **Anti-bucle de auto-respuestas:** IF Auto-reply descarta asuntos con "Auto-reply", "Out of office" o "Undeliverable", y remitentes que contienen `no-reply`, `noreply` o `alerts`. Los descartes se avisan a Slack.
2. **Validación de payload:** Set Payload extrae la dirección del remitente y limita el cuerpo a 2000 caracteres. IF Email Valido verifica que haya remitente.
3. **Look up antes de crear:** HubSpot Look up busca el contacto por email. HubSpot Create solo se ejecuta si el contacto no existe, lo que evita duplicados (Error 409).
4. **Human-in-the-loop:** Gmail Create Draft guarda el borrador con destinatario y no envía nada. Una persona lo revisa y lo envía a mano.
5. **Payload limpio antes de Slack:** Edit Fields deja solo `to` y `subject` antes de publicar en Slack.

## Cómo importar

1. En n8n: Menú → **Import from File** y elegí `checkpoint4_NahuelYair.json`.
2. Asigná las credenciales de Gmail, HubSpot y Slack en los nodos correspondientes.
3. En los tres nodos de Slack, elegí el canal de operaciones desde la lista.
4. Verificá que el trigger esté sin datos fijados y ejecutá los nodos con **Execute step** para comprobar la conexión.

## Conexiones activas

![Credenciales activas](docs/credentials.png)

## Pruebas

- **Ejecución completa:** el flujo se ejecutó de punta a punta con correos reales, desde Gmail Trigger hasta Slack Notificar.
- **Borrador:** se crea con destinatario y sin envío.
- **Auto-respuesta:** un correo "Out of office" se cortó en la rama true de IF Auto-reply, se avisó en Slack y no generó borrador.
- **Duplicado:** un segundo correo desde la misma dirección devolvió el contacto existente en HubSpot Look up, IF Contacto Existe salió por true y HubSpot Create no se ejecutó. En HubSpot hay un solo contacto con ese email.
- **Mensaje final en Slack:** muestra remitente y asunto limpios.
- **No realizada:** la rama de remitente vacío (IF Email Valido en false y Slack Descartado Remitente). No llegan correos sin remitente, por lo que no se generó un caso de prueba para ella.

## Privacidad

El JSON no contiene tokens ni contraseñas: las credenciales se asignan por nombre al importar. El trigger se exporta sin datos de prueba fijados. Las direcciones personales de las capturas están tapadas.
