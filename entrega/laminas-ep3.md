# Las 6 láminas de EP3, por caso

Máximo 6, 4 minutos, y sirven al **IE3** (EP3 §5.2): el caso, la decisión de negocio que apoya el modelo,
la arquitectura desplegada y por qué ese diseño ayuda a decidir mejor. **No repitan lo que van a mostrar
en vivo** (el front, el dashboard, CloudWatch). Archivo: `EP3_Presentacion_Equipo<N>.pdf`, subido a AVA
antes del turno.

Reglas de forma que ayudan en 4 minutos: **un mensaje por lámina** (es el título, en una frase), un número
grande por lámina, y menos de 30 palabras de texto. Lo demás lo dicen ustedes.

Los números de ejemplo son los medidos con esta plantilla; **pongan los suyos** (`herramientas/evidencias.sh`,
`herramientas/costos.py`, `METRICAS.md`).

---

## La misma estructura para los tres

| # | Título (el mensaje) | Contenido | ~Tiempo |
|---|---|---|---|
| 1 | El problema, con su número | Empresa, quién sufre el problema y cuánto cuesta | 30 s |
| 2 | La decisión que cambia | Quién decide, qué decide, con qué salida del modelo | 45 s |
| 3 | El modelo, contra la línea base | Qué predice, con qué datos, una métrica contra la línea base, el costo de equivocarse | 45 s |
| 4 | La arquitectura en AWS | El diagrama de despliegue (capítulo 13.1) | 45 s |
| 5 | Por qué este diseño decide mejor | Reglas antes del modelo, reserva, explicación, LLM fuera del camino; el RNF y su medición | 45 s |
| 6 | Costo y lo que falta | Costo mensual vs presupuesto; la deuda declarada | 30 s |

---

**Abre tu caso.** Los números son los medidos con esta plantilla en el Learner Lab; reemplácenlos por los
suyos.

<details>
<summary><b>Caso 1 · Nexus Retail · recomendación</b></summary>

1. **«Convertimos el 1,5% de las visitas; la meta es 3,5%.»** De cada 100.000 visitantes compran 1.500.
   Llegar a 3,5% son ~USD 305.000 mensuales de venta incremental.
2. **«Qué 4 a 8 productos ve cada cliente, y qué campaña recibe.»** Tienda (ficha y home) y marketing
   (Mailchimp); la gerenta de e-commerce mira el dashboard.
3. **«En 3 categorías acierta el 80,6% de las veces; los más vendidos, el 47,9%.»** Propensión por usuario
   y categoría, solo con clientes que dieron consentimiento (RN2). ROC-AUC 0,75 contra 0,60 de ordenar
   por popularidad. En SageMaker, PR-AUC 0,134 y recall 0,63 tras corregir el sobreajuste.
4. **El diagrama** (la ruta propia `POST /api/campanas/{usuario}`).
5. **«Nunca un producto sin stock, nunca sin consentimiento, siempre con motivo.»** RF5, RN2, RN3; los
   más vendidos si el modelo no responde en 200 ms (RNF3); el LLM redacta la campaña por la cola sin
   datos personales (RNF7). **RNF1 p95 < 200 ms: 113 ms en el dominio** (270 ms desde Chile).
6. **«USD 42 al mes, el 1,9% de lo que queda del presupuesto.»** Deuda: puerto 22, una sola AZ, rate limit
   en memoria; falta la conversión atribuida (RF10).

</details>

<details>
<summary><b>Caso 2 · Andesnet · chatbot</b></summary>

1. **«340.000 contactos al mes, 8 minutos de espera y 22% de abandono.»** El 80% son consultas repetitivas.
2. **«Responder con el artículo correcto o derivar a tiempo.»** El cliente recibe el artículo o el aviso de
   derivación; el ejecutivo ve el historial; Experiencia de clientes mide la derivación (RNF9).
3. **«Con frases nuevas acierta 11 de 20: por eso derivar no depende del modelo.»** Reconoce 10
   intenciones; el F1 de 1,0 del conjunto de prueba está inflado (DESAFÍO M4) y lo dicen.
4. **El diagrama** (la ruta propia `POST /api/conversacion` y la anonimización antes de la cola).
5. **«Pedir una persona funciona siempre, aunque todo lo demás falle.»** Reglas de derivación antes del
   modelo (RF8, RNF11); umbral de confianza; ningún RUT, correo ni tarjeta sale a NVIDIA (RNF5).
   **RNF1 p95 < 3 s con el LLM: 1,79 s sin carga; bajo carga, un worker no alcanza** (y la propuesta).
6. **«USD 193 al mes; 153 son del LLM.»** El costo lo manda el LLM, no la infraestructura: reutilizar el
   texto cuando se repiten intención y artículo, o un modelo más chico, y su efecto.

</details>

<details>
<summary><b>Caso 3 · Kipu Pagos · fraude</b></summary>

1. **«Bloquear clientes buenos nos cuesta más que el fraude.»** Falsos positivos: USD 1,16 M al año; fraude:
   USD 0,5 M. 200.000 transacciones diarias.
2. **«Cada transacción sale aprobada, en revisión o bloqueada, con sus razones.»** El core de pagos decide
   en línea; los seis analistas revisan la cola priorizada por riesgo y monto (RF1, RF2, RF5).
3. **«PR-AUC 0,95 contra 0,11 del azar.»** XGBoost en SageMaker, entrenado solo con las transacciones
   revisadas y sin las columnas que filtran la respuesta. En prueba: 5 falsos positivos y 4 falsos
   negativos en 308. El accuracy no decide: «nunca es fraude» da 99,988%.
4. **El diagrama:** EC2 (Caddy, gateway, dominio, 2 réplicas de IA, base, RabbitMQ, worker) → SageMaker
   serverless; worker → NVIDIA; logs → CloudWatch.
5. **«La decisión nunca espera a nadie que no sea el modelo, y ni a él más de 300 ms.»** Lista negra y
   reglas duras antes del modelo (RF4); umbrales en configuración (RF3); reserva si SageMaker no
   responde (RNF3); el LLM redacta para el analista, por la cola. **RNF1 p99 < 100 ms: no se cumple** con
   serverless (p95 102 ms en el dominio): la propuesta y su costo.
6. **«USD 77 al mes, el 1,3% del presupuesto.»** Deuda: puerto 22 abierto para el despliegue; una sola AZ;
   rate limit en memoria.

</details>

---

## Antes de exportar

- [ ] Cada título es una frase con el mensaje, no un tema («Arquitectura» no; «La decisión nunca espera
      al LLM» sí).
- [ ] Todos los números son de **su** sistema y coinciden con el informe.
- [ ] Ninguna lámina muestra lo que la demo va a mostrar.
- [ ] Ensayado con cronómetro: 4:00 o menos.
