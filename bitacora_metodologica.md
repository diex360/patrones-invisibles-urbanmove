# Bitácora metodológica — Patrones Invisibles (UrbanMove)

## D-01 — Columna vertebral narrativa (Etapa 1)
- Pregunta madre: ¿dónde y cuándo posicionar conductores para minimizar espera sin sobre-ofertar?
- R1 = cuánta incertidumbre hay · R2 = dónde está · R3 = de qué tipo es · R4 = qué hacer.

## D-02 — Entorno (Etapa 2)
- Código ejecutado en el entorno de trabajo con Claude; notebooks portables para GitHub/Colab.
- Solo train.csv (test.csv no trae trip_duration).

## D-03 — Semilla (Etapa 2)
- random_state = 42 en todo proceso aleatorio.

## D-04 — Limpieza (Etapa 4)
- Principio: se eliminan errores de medición, no eventos reales.
- C1 caja NYC · C2 pasajeros 1–6 · C3 duración 60 s–3 h · C4 distancia ≥ 0.1 km · C5 velocidad ≤ 100 km/h.
- 1,458,644 → 1,438,943 (−19,701; −1.351%).
- Viajes >22 h: 1,902 de 1,906 del vendor 2 (99.8% vs. 53.5% base).
- Sesgo horario medido: tasa de eliminación 1.15% (9h) a 2.63% (4h); cambio máx. en curva horaria 0.014 pp.
- Trade-off reconocido: C4 elimina 5,897 viajes con distancia 0 (duración mediana 5.8 min), probablemente reales con GPS de destino fallido.

### REGLA DE DOCUMENTACIÓN — días atípicos (confirmada por Diego)
Dondequiera que se mencionen (PDF, notebook, video, defensa) debe aclararse:
1. Son eventos reales (23–24 ene 2016; 30 may 2016 Memorial Day), no errores.
2. Se conservan deliberadamente por coherencia con el principio de limpieza.
3. Están marcados con la columna `dia_atipico` (decisión reversible).
4. Pesan 10,436 viajes = 0.725% del dataset limpio.
5. La atribución del 23–24 ene a una ventisca debe verificarse con fuente externa antes de afirmarse.

## D-05 — Muestreo (Etapa 5) — CONFIRMADA por Diego
- Estrategia: estratificada proporcional por día×hora (168 estratos), n = 50,004, semilla 42.
- Motivo del muestreo: costo de Roles 2 y 3 (silhouette ~n²: 7.4 s a 30k, 19.7 s a 50k, 78.3 s a 100k).
- Representatividad (KL muestra‖población, bits): hora 0.00000 · día 0.00000 · mes 0.00013 · zona 0.00667 · duración 0.00042.
- Zonas: 250 de 620; cubren 99.851% de la demanda. Días atípicos en muestra: 356 (0.712%).
- Alternativas evaluadas (5 semillas): aleatoria simple, estratificada por mes, filtro a un mes.
- Filtro a un mes descartado: KL día 0.0042–0.0098 bits.
- Borrador de justificación PDF: 186 palabras (límite 200).
- Énfasis de Diego: lo que se defiende es la cadena de porqués, no la cifra.
- Cadena: (1) costo de R2/R3 → (2) misma muestra para integrar → (3) estratificar día×hora hace exacto el eje del R1 → (4) 50k = rendimientos decrecientes en error espacial → (5) representatividad medida con KL.
- Debilidad reconocida: silhouette admite submuestreo (sample_size); respuesta: EM manual y DBSCAN no, y la coherencia entre roles exige una muestra única.

## D-06 — Definición de zona (Etapa 6) — PROPUESTA
- Celda cuadrada de 1 km sobre coordenadas proyectadas (equirectangular local, centro lat 40.7510, lon −73.9737).
- Error de la proyección vs Haversine: mediana 0.11 m, p99 11.76 m, máx 57.53 m (0.20%).
- En grados crudos, 1 km E-O pesa 1.32x más que 1 km N-S (1° lat = 111.2 km; 1° lon = 84.2 km).
- Sensibilidad a resolución: H(zona recogida) = 8.939 (0.25 km) · 7.208 (0.5) · 5.461 (1) · 3.845 (2) · 2.962 (3) bits.
- Sensibilidad al origen del grid (1 km): H entre 5.456 y 5.510 bits.
- Operativo: velocidad mediana 12.87 km/h → 1 km ≈ 4.7 min.
- Zonas: 242 de recogida, 552 de destino (en la muestra).
- Alternativas descartadas: grid en grados (distorsión 1.32x), zonas oficiales TLC (áreas desiguales + archivo externo), K-means preliminar (circular con Rol 2).

## D-07 — Resolución temporal — ABIERTA (se decide en Etapa 8)
- Sesgo aprox. del estimador plug-in de I(hora; zona) ≈ (Kz−1)(Kh−1)/(2n ln2).
- Recogida (Kz=242): 0.0800 bits con 24 h; 0.0174 con 6 franjas; 0.0028 con 24 h en población.
- Destino (Kz=552): 0.1828 bits con 24 h; 0.0397 con 6 franjas; ~0.0064 con 24 h en población.

## D-08 — Variables (Etapa 6) — PROPUESTA
- Se crean: dist_km (Haversine), hour, dow, is_weekend, px/py/dx/dy (km), zona_pick, zona_drop, log_dist, log_dur, speed_kmh (solo interpretación).
- Se excluyen del modelado: store_and_fwd_flag (266 'Y' = 0.53%, indicador técnico); passenger_count (70.4% = 1, discreta; solo descriptiva).
- vendor_id: solo para una posible comparación distribucional del Rol 1.

## D-09 — Transformaciones (parcial, Etapa 6)
- Log en distancia y duración: asimetría 2.800→0.359 y 2.374→−0.123.
- log_vel excluida de cualquier modelo conjunto: log_vel = log_dist − log_dur exacto (autovalor 0, rango 2 → covarianza singular en GMM).

### CORRECCIÓN (Etapa 7) — constantes unificadas en config.py
- Causa: la Etapa 6 usó el centro de proyección con decimales completos; config.py lo fija a 4 decimales. 0.45% de viajes en bordes cambiaba de celda entre muestra y población.
- Solución: todo script lee config.py. Coincidencia muestra↔población: 100%.
- Valores OFICIALES a 1 km: zonas recogida 242, destino 553; H(zona recogida) = 5.462 bits; sesgo aprox. I(hora; destino) 24 h = 0.1832 bits.
- La tabla de sensibilidad por resolución (Etapa 6) queda como exploratoria (tendencia), no como cifra oficial.
- Población con el mismo grid: 603 zonas de recogida, 1,123 de destino.

## D-07 — Resolución temporal y sesgo (Etapa 8) — RESUELTA (propuesta)
- Se mantienen 24 horas (lo que pide la guía). El sesgo se mide por permutación (200 permutaciones de la hora).
- I(hora; destino) muestra: observado 0.2167, nulo medio 0.1177 (54% del observado es sesgo), neto 0.0991; población 0.1144.
- I(hora; recogida) muestra: observado 0.1305, nulo 0.0571, neto 0.0734; población 0.0813.
- Fórmula Miller-Madow sobreestima el sesgo (0.1831 / 0.0800) frente a la permutación.
- Política propuesta: cantidades globales del Rol 1 (conteos, baratos) se reportan sobre la POBLACIÓN; la muestra se usa como verificación y para cruces por viaje. Coherente con Etapa 5: el muestreo lo exigen R2/R3, no R1.

## ROL 1 — Resultados oficiales (población salvo indicación)
1. Probabilidad: top 10 zonas = 46.87% de recogidas (muestra); 361 zonas con P_MLE=0 en muestra pesan 0.147% → suavizado aditivo.
2. Entropía: H(recogida) 5.466 bits (muestra 5.462) → 2^H ≈ 44 zonas efectivas de 242; H(destino) 6.035 (muestra 6.016) → ≈ 65 de 553.
   Por hora: más predecible 3h (5.195 bits, 36.6 zonas ef.); menos 5h (5.696, 51.8). Pico 20–21h: 5.310 (39.7).
   Plug-in en muestra subestima −0.065 bits en promedio; con Miller-Madow −0.023.
3. Entropía condicional: H(recogida|hora) 5.385; H(destino|hora) 5.920.
4. Información mutua: I(hora;recogida) 0.0813 (1.49% de H); I(hora;destino) 0.1144 (1.90%). p < 0.005 (permutación).
   Identidad verificada: Σ p(h) KL(P(Z|h)‖P(Z)) = 0.1144. KL destino por hora: máx 4h 0.538; mín 17h 0.038.
5. KL laboral vs finde: HORA 0.1815 (finde‖lab) / 0.1609 (lab‖finde); ZONA recogida α=0.5: 0.0549 / 0.0503; ZONA destino: 0.0398 / 0.0390.
   Ruido de referencia (muestra): hora 0.0013; zona recogida 0.0186. Sensibilidad α∈{0.1,0.5,1} (muestra, recogida): 0.0654–0.0605.
6. Entropía cruzada: planificar finde con modelo laboral → +0.1815 bits en hora (+4.06%); +0.0549 en zona (+0.99%).
7. Jensen: (a) H(Z)−H(Z|hora) = I (0.0813 / 0.1144). (b) exp(E[log dur]) 10.84 min vs E[dur] 13.99 min (29.1%); dist 2.29 vs 3.44 km (50.0%) [muestra].
8. Log-sum-exp: clasificación de 182 días (zona×hora, 2,337 celdas, α=0.5, dejar-un-día-fuera): 97.80% (178/182) vs 71.43% base.
   Producto ingenuo = 0.0 (log-verosimilitud ≈ −1892 nats ≈ 10^−822). Errores: 2016-01-01 y 2016-05-30 (laborales vistos como finde; festivos); 2016-01-02 (p=0.24) y 2016-01-24 (post-ventisca) vistos como laborales.
   Limitación: supuesto de independencia entre viajes → posteriores extremos (sobreconfianza); válido para clasificar, no para leer la probabilidad literal.

## ROL 2 — Clustering (Etapa 9)
Espacio: recogidas (px, py) km, sin escalar, n = 50,004.
### K-means — selección de K (K=2..20; n_init=10; estabilidad = ARI mínimo contra 5 semillas)
- K=2: silueta 0.809 pero TRIVIAL (separa JFK, 2.21%, del resto) → se descarta.
- Silueta máx. no trivial: K=4 (0.492; DB 0.502). K=5: 0.452. Codo (máx. distancia a la cuerda): K=5.
- Estabilidad: ARI ≥ 0.96 para K ≤ 8; K=9 colapsa (0.523); K ≥ 13 inestable (< 0.8 salvo casos).
- Calinski-Harabasz crece casi monótono → no discrimina.
- DECIDIDO (delegado por Diego, con justificación): K=5 — KM0 Midtown 44.08% · KM1 Downtown 27.58% · KM2 Upper Manhattan 23.11% · KM3 LaGuardia 3.06% · KM4 JFK 2.17%.
  Radios medianos Manhattan 1.10–1.41 km (compatibles con escala de 1 km de D-06) vs K=4: p90 2.5–2.8 km.
### DBSCAN — min_samples=20 (≈3.2 recogidas reales/día en el radio), ε=418 m (rodilla k-distancia, p98.43)
- 10 clusters; ruido 1.034% (517 viajes ≈ 81.7 recogidas reales/día).
- DB0 = 92.301% (masa continua de Manhattan) · DB1 LaGuardia 2.458% · DB2 JFK 2.060% · hubs secundarios fuera de Manhattan (DB3 ≈72/día, DB4 ≈38/día, DB6 ≈22/día, DB5 ≈15/día…). Nombres de barrios A VERIFICAR en mapa.
- Sensibilidad (ms=20, ε 300–600 m): ruido 1.92%–0.67%; clusters 11–8. ms=10 produce 24 clusters (mínimo 7 viajes: micro-clusters espurios).
- Silueta sin ruido ≈ 0.04: NO es fallo; la silueta supone clusters convexos y DB0 es una masa alargada continua.
### Comparación
- ARI K-means vs DBSCAN (sin ruido) = 0.100: responden preguntas distintas.
- Ambos aíslan los aeropuertos. DBSCAN muestra que Manhattan es un continuo → los cortes de K-means son operativos, no naturales.
- K-means esconde hubs de Brooklyn dentro de KM1 (Downtown Manhattan).
### Ruido
- Vecinos en 418 m: mediana 4 en muestra ≈ 0.63 recogidas reales/día → 1 cada 37.9 h.
- vs no ruido: madrugada 0–5h 32.50% vs 11.48%; finde 36.94% vs 28.43%; dist 3.22 vs 2.10 km; destino en top-20 zonas 17.41% vs 65.15%; a 8.17 km del centro vs 2.40.
- Ruido dentro de cada KM: KM3 10.31%, KM4 5.07%, KM1 1.54%, KM2 0.61%, KM0 0.10%.
- Puente R1: la madrugada es la hora más dispersa (5h ≈ 52 zonas ef.) y con destino más distinto (KL 4h = 0.538).
### D-13 — DECIDIDA: K=5; DBSCAN min_samples=20, ε=418 m
- Cadena K: (1) descartar K=2 (trivial) y K inestables → (2) silueta dice 4, codo dice 5 → (3) desempate operativo: radio 1.1–1.4 km vs 2.5–2.8 km → (4) costo explícito: −0.040 de silueta.
- Cadena DBSCAN: min_samples=20 ≈ 3.2 recogidas reales/día; 10 → micro-clusters espurios; 50 → demasiado estricto; ε por rodilla; ruido ~1% robusto en ε 300–600 m.

## ROL 3 — Mezclas gaussianas y EM (Etapa 10)
Espacio: (log_dist, log_dur), n = 50,004, covarianza completa, reg_covar 1e-6.
### CORRECCIONES (resultados preliminares invalidados — NO usar)
- Con tol=1e-6 se afirmó: "K=5 tiene soluciones distintas no identificables" y "mínimo de BIC en K=5". AMBOS FALSOS: artefacto de parada prematura en una meseta.
- Se dijo que 1e-6 es el default de sklearn: FALSO. El default es tol=1e-3 y max_iter=100 (aún más laxo).
### Criterio de convergencia (D-15)
- |Δ log-verosimilitud media| < 1e-9 entre iteraciones. Verificado: con 1e-6, tres semillas de K=4 paraban en −1.79289/−1.79292 con pesos distintos; con 1e-9 las tres llegan a −1.792874.
### Selección de K (BIC con tol=1e-9, k-means++)
- K1 186,623.1 · K2 182,761.4 · K3 181,161.3 · K4 179,550.6 · K5 179,390.0 · K6 179,315.7 (semilla 42) / 179,430.2 (semilla 43) · K7 179,472.3 (7,759 iter).
- K8 NO recalculado con tol estricto (laxo: 179,539.5).
- DECISIÓN K=5: (1) BIC cae fuerte hasta K=4, luego −161 (K5) y −74 (K6, solo en el mejor caso), sube en K7; (2) K=6 NO es reproducible: dos semillas → dos soluciones distintas con tol estricto (ll −1.789227 vs −1.790372), una peor que K=5 → mínimos locales REALES; (3) K=5 converge al mismo óptimo desde 10 inicializaciones; (4) el componente extra de K=5 es interpretable (largo interdistrital nocturno).
### Inicializaciones (EM manual, K=5, tol 1e-9, 5 semillas c/u)
- Las 10 corridas → mismo óptimo ll = −1.790619. EM manual = sklearn (mismos valores).
- Iteraciones: k-means++ 1,266–2,947 (1,395 con semilla 42); aleatoria 4,345–4,780.
- Historia semilla 42: distancia al final <1e-3 en iter 58 (k-means++) vs 2,837 (aleatoria); <1e-4: 142 vs 3,135; <1e-5: 534 vs 3,611. Log-verosimilitud monótona creciente en ambas (garantía del EM).
- Con tol=1e-6 (preliminar), la aleatoria paraba en puntos peores 4/10 veces en K=5: riesgo práctico con criterios laxos.
### Modelo final (P0..P4 por distancia creciente; valores típicos = medias geométricas)
- P0 π=0.0280 · 0.63 km · 7.2 min · 5.3 km/h · Σ=[[0.5207,0.2756],[0.2756,0.6830]]
- P1 π=0.4206 · 1.29 km · 7.0 min · 11.0 km/h · Σ=[[0.2499,0.2133],[0.2133,0.4105]]
- P2 π=0.4046 · 2.78 km · 12.7 min · 13.1 km/h · Σ=[[0.2601,0.1689],[0.1689,0.2394]]
- P3 π=0.1285 · 8.09 km · 24.2 min · 20.0 km/h · Σ=[[0.2024,0.1130],[0.1130,0.1801]] · 0–5h 19.8% (base 11.7%), hora modal 23h, 20.9% desde zona LGA
- P4 π=0.0183 · 20.49 km · 44.7 min · 27.5 km/h · σ log dist 0.0495 (×1.05) · 59.4% desde zona JFK · r_max media 0.866
- N_k efectivos = [1,399.6; 21,029.7; 20,233.4; 6,426.8; 914.5]
### Responsabilidades
- r_max mediana 0.754; ≥0.9: 11.74%; <0.6: 18.19%. Entropía media 0.857 bits (máx 2.322).
- Certeza por zona: JFK 0.867 (5.5% ambiguos), LGA 0.821 (6.9%), Manhattan 0.731–0.738 (~18–19%). Por franja: plana (0.735–0.752).
- Ejemplo paso E: id0710740 (2.25 km, 18.8 min): r = [0.0085, 0.4368, 0.5500, 0.0047, 0.0000]; término P4 = −1023.4 (e^−1023 no representable).
### Cruces
- No aparece perfil "commuter de hora pico": 7–9h por perfil 10.7–14.2% vs 13.1% base; finde 26.4–30.2% vs 28.5%. Coherente con R1 (la hora aporta poca información).
- P1/P2 se solapan (continuo), como Manhattan en R2.

## ETAPA 11 — Integración (tabla maestra 50,004 viajes, 30 columnas)
### P1: zona (R2) vs hora (R1) para explicar el TIPO de viaje (R3)
- H(perfil) marginal = 1.5988 bits (máx 2.3219).
- I(zona_R2; perfil) neto (perm.) = 0.1389 bits = 8.69% de la incertidumbre del tipo.
- I(hora; perfil) neto = 0.0199 bits = 1.24%. Zona explica ~7x más que hora.
- Composición por zona: LaGuardia 78.3% P3 (largo); JFK 54.9% P4 + 38.0% P3; Manhattan (KM0/1/2) 90%+ en P1+P2.
### P2: fin de semana — tipo de viaje casi no cambia
- KL(finde‖laboral) en perfil (duro) 0.0037; ponderado 0.0020 bits. Comparar: KL en HORA (R1) = 0.1815; en ZONA = 0.0549.
- Cambio real: P1→P2 (−1.84pp / +2.22pp), viajes ligeramente más largos el finde. P3/P4 casi iguales.
- Por franja: P3 (largo) sube de ~11% (día) a 20.1% (madrugada 0–5h) y 16.2% (noche 21–23h).
### P3: el ruido DBSCAN es demanda de otro TIPO, no solo periférica
- Composición (ponderada): ruido tiene 1.79x más P3 y 1.91x más P4 que lo normal; 26.2% del ruido es P3+P4 vs 14.6% normal.
- Entropía de la mezcla: ruido 1.836 bits vs normal 1.682 (más diversa/impredecible en tipo, no solo en ubicación).
### Puente de 3 patas — madrugada
- % ruido DBSCAN: madrugada (0–5h) 2.87% vs resto 0.79% (3.6x).
- Perfil P3 (largo) en madrugada 20.1% vs 11.9% resto.
- Cierra el círculo con R1 (madrugada = hora más dispersa, 5h ≈52 zonas ef.; destino más distinto a las 4h, KL=0.538).
### Cadena aeropuerto
- JFK: H(hora)=4.363 bits (vs ciudad 4.464, similar); ruido 5.07%; perfil 49.5% P4 + 41.9% P3.
- LaGuardia: H(hora)=4.405; ruido 10.31% (2x JFK); perfil 68.4% P3 (no tiene un perfil "propio" como JFK P4).
- Lectura: JFK es una demanda de firma propia y predecible en tipo (aunque no más predecible en hora); LaGuardia es más ruidosa y se confunde con "viaje largo genérico".

## Equipo (confirmado por Diego)
- Diego Alejandro Sandoval — 339271 — Rol 4
- Daniel Figueredo — 332679 — Rol 1
- Juan Esteban Ocampo — 286388 — Rol 2
- Juan Andres Martinez — 326771 — Rol 3

## Etapa 13 — Documento PDF generado
- /mnt/user-data/outputs/documento_soporte.pdf (5 páginas, reportlab). Pendiente: enlace GitHub (Etapa 14).
